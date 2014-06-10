nuxeo-platform-mail-automation
==============================

## What is this project ?

This module is an addon for `nuxeo-platform-mail` module that provides the `MailFolder` feature.

The goal of this module is to integrate Nuxeo Automation with `nuxeo-platform-mail`

## Why would you use this ?

The MailFolder create Document from Mail using a configured chain of actions.

The extension point is called actionPipes.

The default configuration contains :

      <action>
        org.nuxeo.ecm.platform.mail.listener.action.StartAction
      </action>
      <action>
        org.nuxeo.ecm.platform.mail.listener.action.ExtractMessageInformationAction
      </action>
      <action>
        org.nuxeo.ecm.platform.mail.listener.action.CheckMailUnicity
      </action>
      <action>
        org.nuxeo.ecm.platform.mail.listener.action.CreateDocumentsAction
      </action>
      <action>
        org.nuxeo.ecm.platform.mail.listener.action.EndAction
      </action>

The action that will actually create the Document is `CreateDocumentsAction`.

So, if you want the mail folder to create custom Document types, you have to contribute your own Action and overriding the default chain.

 - this may seem more complex than actually needed (even if the code itself is rather simple)
 - today, all this configuration would be done via Automation

The goal of this addon is to do the heavy lifting and basically "make a new Mail Action that uses an Automation chain to do the creation".

This will allow you to define everything from within Nuxeo Studio.

## Configuring the Automation chain

The new contributed MailAction will use a Chain to do the work.

The target chain will be lokked up by name :

 - using the name defined in the Nuxeo system property `org.nuxeo.mail.automation.chain`
 - defaulting to `CreateMailDocumentFromAutomation`

NB : in 5.8, we don't have a better way to handle this ...

The Chain itself should be something like : 


    <chain id="CreateMailDocumentFromAutomation">
      <operation id="Context.RestoreDocumentInput">
        <param type="string" name="name">mailFolder</param>
      </operation>
      <operation id="Document.Create">
        <param type="string" name="type">MailMessage</param>
        <param type="string" name="name">expr:Context["mailDocumentName"]</param>
        <param type="properties" name="properties">expr:dc:title=@{subject}
    mail:messageId=@{messageId}
    mail:sender=@{sender}
    mail:recipients=@{recipients}
    mail:cc_recipients=@{ccRecipients}
    mail:htmlText=@{text}</param>
      </operation>
      <operation id="Context.SetInputAsVar">
        <param type="string" name="name">mailDocument</param>
      </operation>
      <operation id="Context.RunOperationOnList">
        <param type="string" name="id">ProcessAttachment</param>
        <param type="string" name="list">attachments</param>
        <param type="boolean" name="isolate">false</param>
        <param type="string" name="item">attachment</param>
      </operation>
      <operation id="Context.RestoreDocumentInput">
        <param type="string" name="name">mailDocument</param>
      </operation>
      <operation id="Document.Save"/>
    </chain>


    <chain id="ProcessAttachment">
      <operation id="Context.RestoreBlobInput">
        <param type="string" name="name">attachment</param>
      </operation>
      <operation id="Blob.Attach">
        <param type="document" name="document">expr:Context["mailDocument"]</param>
        <param type="boolean" name="save">false</param>
        <param type="string" name="xpath">files:files</param>
      </operation>
    </chain>


## Building

    mvn clean install 

## How to use it 

...
