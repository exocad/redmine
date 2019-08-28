# Exomine Upgrade Report (2.5 -> 4.0)

## Redmine

For Testing, disable SMTP in config/configuration.rb and use `delivery_method :file` - (theoretically) sent mails can then be found in tmp/mails as text-files.

__Not verified__: Added `own_watch`/`own_watch_contributed` to issue visiblity [010eee1066cc3c67abfcebe2c826a66fc63b2542]

Related files:
* app/models/issue.rb
* app/models/role.rb
---

Adjust mail subjects [ca486dd3799a98b249cfc577b309fdbe2ed5d7b4]
* app/models/mailer.rb
---
Change mail-header `X-Auto-Response-Suppress` from `'OOF'` to `'All'` [79fc2f696be2dedba2255700d8f5f6625ac74a60]

Verification: Update an arbitrary issue, check headers of notification-email

**This does not apply to mails sent via helpdesk plugin (i.e. via "SEND E-MAIL TO CUSTOMER"-Button)**
* app/models/mailer.rb
---
Send issue-notifications only to users who are author or actual assigned to an issue, but not former assignees or in the same group [010eee1066cc3c67abfcebe2c826a66fc63b2542]

Verification:
* If assignee is a whole group, notification should only be sent to the issue's author (if exists)
* If assignee is an user, notification should only be sent to new asignee and issue's author (if exists)
* Former assignees never should be notified.

Related files:
* app/models/user.rb
---
Submit button on issues now labelled with "Submit as INTERNAL note", adding placeholder for helpdesk-plugin to inject "SEND E-MAIL TO CUSTOMER" button. CSS classes added to fit exocad-theme [ad47ce6acd7f4356b95d655674c4ffaad76ad147]

Verification:
* Open arbitrary issue, click Update, check the submit-button.

Related files:
* app/views/issues/_edit.html.erb
---
Serve exomine in `<host>/exomine/`-subdirectory. Former adjustment now obsolete, now done via environment-variable: In apache's vhost-configuration within the \<directory\>-directive use `SetEnv RAILS_RELATIVE_URL_ROOT /exomine` [e1f6643a65c8653f71a31f371ea6042534f1161e]

Related files:
* config/environment.rb

---
Adjust title of pasted files [561195a25f958ca9d888e2c932954ed419c7a626]

__Caution__: As of redmine 4.1 pasting images will be [natively supported](http://www.redmine.org/issues/3816). As of now, the new version does not preview the pasted screen and also cropping is not available. This easily can cause information leakage (e.g. when <kbd>AltGr</kbd> + <kbd>Print</kbd> on a wrong-focused window). Consider disabling the native feature and using the clipboard-image-paste plugin instead. The exocad-fork of the plugin works with redmine 4.x.

Related files:
* clipboard_image_paste/assets/javascripts/clipboard_image_paste.js
---
Check all mails, including seen but not deleted ones [737970976b11291c8a88fa93cd434d21dc4b10f3]

Verification:
* Send an email to a [test-]mailbox and mark it as "seen", then let use helpdesk's settings (in project) check the mailbox: See if it is processed.

Related files:
* redmine_contacts/utils/check_mail.rb
---

Adjustments on Helpdesk plugin
* __Obsolete__: initial_message also include ticket number in subject, is now in *helpdesk_mail_messenger/initial_message.rb* implemented by vendor [5c1974da1b4018974294f5d66fcb67924c677c81]
* __Critical__: Send original ticket description <u>only</u> if initiated from customer's email to prevent leaking internals to the customer (now in *helpdesk_mail_support.rb*) [7635464630842060121baa8da75607e3e90ee711]
    * Create an issue within a project, having helpdesk- and contacts-plugin enabled.
    * Update ticket as internal note
    * Update ticket with "send e-mail"
    * Check contents of the sent mail, make sure the internal note is not included in the ticket history.
* Customize quoted text to be properly detected by mail-clients (now in *helpdesk_mail_messenger/initial_message.rb*) [7635464630842060121baa8da75607e3e90ee711]
Add people to future conversation which were not involved so far (now in *helpdesk_mail_recipient/issue_reply_recipient.rb*) [c7f93f4dc023bb1823aeae956c0447661df62531]
    * Create ticket with helpdesk-contact (other than author, i.e. alice@foo.com)
    * Forward ticket-e-mail to a second e-mail-account (bob@bar.com)
    * Respond to ticket from the second account
    * When now updating the ticket: The "To"-contact should now be the second address (bob@bar.com), the "CC"-contact the first address (alice@foo.com).
* For created contacts only add domain-name as company (instead of whole mail-address; now in *helpdesk_mail_support.rb*) [0ac65abe044dcaa727b478025ce88d9fc5ebf9ff]
* Add created contacts to all projects (now in *helpdesk_mail_support.rb*) [210ef909b81508dfe5dc86cd07e4db42cbedd74a]<br>
__Heads up__: On admin-panel there is a new setting for the contacts-plugin introduced, found in tab "Auto assignment". Contacts created within one of the activated projects here are assigned to all projects. Newly created projects are disabled by default.
    * create two test-projects, enable auto-assignment for only one of them
    * create one contact within each project
    * the project with auto-assignment then should contain only its own contact, the project without auto-assignment should contain both contacts.
* support@exocad.com always in BCC [8be9f4d600b6b1118c91120cef2637c19cba5dbd]
* __Obsolete__: auto hide bcc field when toggling send mail (BCC always visible, send-mail-checkbox hidden) [c65255a7dc64dd350b78d3707b3b9e0f768f6264]
* __NEED REVIEW__: include all CC recipients by default in all responses [8e1e606f80d083ee7f477e2d7385e2748101b148]
* Use "Submit as INTERNAL note"-button instead of checkbox [c6e85457a30bab089bb442e1f6f2ff6398641721]
    * Create ticket for test-customer ("Send as"-field should be set to "(Dont't send)")
    * Update ticket, using internal-note-button
    * Verify that test-customer has not any e-mail related to the ticket.
* Use "Send to customer"-button instead of checkbox [a3f3bae0185b8c6807d9769ae785f4a3eaebe172]
    * Create ticket for test-customer
    * Update ticket, using "sent to customer"-button
    * Verify that test-customer has received the e-mail containing the update.
* Remove unneccessary fields on ticket createn (ticket_date, ticket_time) [520d068abefdf8250fe63ca92bee091df244279b]
* Helpdesk ticket allow for any type of ticket tracker even if HelpdeskSettings.helpdesk_tracker is not set to "all" (According to commit-msg this _should be discussed if really required_) [47a413c5814302f319dbc2fd566b25ff651340e5]
* consider all to-addresses instead of only the first when processing emails with multiple recipients [e7018600cce7787a40e7180a84b5583030fe82b5]


Related files:
* redmine_contacts_helpdesk/app/models/helpdesk_mailer.rb
* redmine_contacts_helpdesk/app/models/helpdesk_mailer.rb
* redmine_contacts_helpdesk/app/models/helpdesk_mail_support.rb
* redmine_contacts_helpdesk/app/models/helpdesk_mail_messenger/initial_message.rb
* redmine_contacts_helpdesk/app/models/helpdesk_mail_recipient/issue_reply_recipient.rb
* redmine_contacts_helpdesk/app/views/issues/_send_response.html.erb
* redmine_contacts_helpdesk/app/views/issues/_ticket_data_form.html.erb
* redmine_contacts_helpdesk/lib/redmine_helpdesk/patches/journal_patch.rb

---

Manual tickets defaults source "By Phone" instead of "By Email" [93dacad7ef5e155936198c91b9dc66010fac8f77]

Verify:
* creating an issue within a project with contacts- and helpdesk-plugin enabled, should show field "Ticket source" with "Phone" selected.

Related files:
* redmine_contacts_helpdesk/app/models/helpdesk_ticket.rb

---

### Misc
* Removed "also available as PDF"-Link (app/views/issues/show.html.erb) [54cb96525b52571665238bbf2167dde39949d30d]
* Remove latest projects from "my"-page (app/views/welcome/index.html.erb) [d63388cd89d69ec4da5911e39d4d9f9e5425e8fe]
* Adjustments in i18n (config/locales; de, en-GB, en) [48f70c720d431b02e551e80bcfd1ef6eb332f4b7]
* limit latest issues to 20 on "Home"-page (latest_issues/lib/latest_issues/view_hook_listener.rb) [b6cb00e8e406c3ff62f42cb8013d50e6350122a1]
