Mailer
======

TreeFrog Framework incorporates a mailer (mail client) which makes it possible to send mail by SMTP. For now (v1.0), only SMTP message sending is possible. To create mail messages an ERB template is used.  

First, let's create a mail skeleton with the following command.

.. code-block:: bash
  
  $ tspawn mailer information send
    created  controllers/informationmailer.h
    created  controllers/informationmailer.cpp
    created  controllers/controllers.pro
    created  views/mailer/mail.erb

The class of InformationMailer is created in the controller directory, and a template with the name *mail.erb* is created in the views directory.

Next, open the mail.erb that was created, and then save as the following content.

::
  
  Subject: Test Mail
  To: <%==$ to %>
  From: foo@example.com
     
  Hi,
  This is a test mail.

Above the blank line is the mail header, and below is the content of the body. Specify subject and destination in the header. It is possible to add any field of header. However, Content-Type and Date field are automatically added, so that you don’t need to write them there.  

If you are using multi-byte characters, such as Japanese, save the file by setting the encoding (default UTF-8) in the settings file of InternalEncoding.

Call the deliver() method at the end of the send() method of InformationMailer class.

.. code-block:: c++
  
  void InformationMailer::send()
  {
      QString to = "sample@example.com";
      texport(to);
      deliver("mail");   // ← mail.erb Mail sent by template
  }

You are now able to call from outside of class. By writing the following code in the action, the process of mail sending is executed.

.. code-block:: c++
  
  InformationMailer().send();


When you actually send mail, please see the following "SMTP Settings" section.

When you need to send mail directly without using a template, you can use the TSmtpMailer::send() method.

SMTP Settings

There is no configuration information for SMTP in the code above. So, set information about SMTP in the *application.ini* file.

.. code-block:: ini
  
  # Specify the connection's host name or IP address. 
  ActionMailer.smtp.HostName=smtp.example.com
   
  # Specify the connection's port number.  
  ActionMailer.smtp.Port=25
    
  # Enables SMTP authentication if true; disables SMTP
  # authentication if false.
  ActionMailer.smtp.Authentication=false
    
  # Specify the user name for SMTP authentication.
  ActionMailer.smtp.UserName=
    
  # Specify the password for SMTP authentication.
  ActionMailer.smtp.Password=
    
  # Enables the delayed delivery of email if true. If enabled, deliver() method
  # only adds the email to the queue and therefore the method doesn't block.
  ActionMailer.smtp.DelayedDelivery=false

If you do SMTP authentication, you specify the ActionMailer.smtp.Authentication=true.
As for the authentication method, CRAM-MD5, LOGIN, PLAIN are mounted so, by this priority, the authentication process is performed automatically.

In this framework, SMTPS email sending is not supported.

Delay sending the mail
----------------------

Since mail sending by SMTP needs to pass the data through an external server, it requires time compared to processes. You can return an HTTP response before the mail sending process is executed. 

Edit the *application.ini* file as follows.

.. code-block:: ini
  
  ActionMailer.smtp.DelayedDelivery=true

By doing this, the deliver() method will be a non-blocking function of merely queuing data. Processing of the mail sending will occur after returning an HTTP response.

**Additional note**
If you do not set the delay sending (in the case of false), the deliver() method would keep blocking until the SMTP processing would end, or be in error.
