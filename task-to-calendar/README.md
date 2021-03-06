# Email calendar entry for task due dates

This is an example of a simple Action Module component using the 'resilient-circuits' framework.


Use Case:
Tasks have due dates.  This custom action processor creates a .ics file with the due date for the task and emails
the file to the owner of the task.  If the task does not have a due date, or there is not an owner,  no action is taken.
When the task does not have a due date, the event is created as a single day event.

This integration can be run on the Resilient appliance or anywhere else.  
The code is written in Python, and can be extended for your own purposes.

Requires:
* Python version 2.7.x or 3.4.x or later.
* Python libraries `resilient` and `resilient-circuits` installed.
* Resilient Server or hosted Application, version 27 or later, with Action Module.

## Resilient customizations

You must configure the following customizations to the Resilient server.
Open the Customization Settings menu, then:

### Message Destination
Open the Message Destinations tab.
Create a Queue message destination with programmatic name `taskcalendar`.
Select Yes for "expect acknowledgement", and add the integration user
to its users list.

![Custom message destination](Documents/messagedestination.png)


### Automatic Rules
Open the Rules tab.
Create two automatic rules named 'taskcalendar' and 'DueDate change', associated with object type
"Task".  Choose `taskcalendar` as the message destination. 

For the rule 'taskcalendar', add condition
"Owner is changed"
![Owner Change custom action](Documents/taskcalendar.png)

For the rule 'DueDate change' add condition
"Due Date is changed"

![Due Date Change custom action](Documents/duedatechange.png)

---

## Python setup

Ensure that the `resilient` and `resilient-circuits` libraries are installed.


### Configuration Parameters

The script reads configuration parameters from a file.
The configuration file is named `app.config`, in the same
directory as the scripts.  Edit this file to provide appropriate values
appropriate for your environment (server URL and authentication credentials).
Verify that the logging directory has been created. To set a different configuration file
set the environment variable 'APP_CONFIG_FILE' to the file you intend to use.

There are two sections to the configuration file.  The 'resilient' section configures the parameters
for accessing the resilient server via the api.  The 'taskcalendar' section configures the action component.
Within the 'taskcalendar' section the following information is required:
* queue - this should be 'taskcalendar' 
* smtpserver - the server which will send the email with the .ics file as an attachment
* smtpfrom - the email address which the email should be shown as from
* smtpuser - login credential for the user on the smtp server. Usually an email address
* smtppw - password for the 'smtpuser' on the smtp server
* smtpport - port on the smtp server.
* use_start_tls - Only include and set to True if using TLS, usually with port 587
* use_ssl - Only include and set to True if using legacy SMTP over SSL, usually with port 465


### Running the example

In the current directory, run the custom action application with:

    resilient-circuits run --componentsdir /<your_full_path>/task-to-calendar/components/

The script will start running, and wait for messages.  When a task is assigned to a user, or when the due date of the task is changed, the
action processor will be invoked.  If the task only has an owner assigned, then the event is considered a single day event. If there is a due date, then the event is created as a multi day event starting with the current day and ending on the due date.  If the due date is changed, a new event is sent, which will replace the original event in the calendar.  If the owner is changed, the new owner will get the invite.  The previous owner will NOT get an update to remove the event from the calendar.


To stop the script running, interrupt it with `Ctrl+C`.

### Limitations

* Some calendar applications will not display the embedded url
* Updating an existing calendar event may not work properly depending on the calendar application you are using
* If you update the Task's owner and due date at the same time in Resilient, the result will be 2 separate identical calendar invites being sent out.


### More
For more extensive integrations with the actions module contact
[success@resilientsystems.com](success@resilientsystems.com).
