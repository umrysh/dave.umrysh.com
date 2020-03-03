---
title: "Make an Active Directory user a local admin"
date: 2020-03-03T10:27:04-07:00
draft: false
---

What follows are the steps I use to make an Active Directory user account a local administrator in Windows 10.

* The first step is to open the control panel. You can search for the app by typing "control panel" in the bottom left search bar.

![Open control panel](/img/open_control_panel.png)

* Click on the option called "User Accounts"

![Open user accounts](/img/open_user_accounts.png)

* In the next screen click on the link titled "Give other users access to this computer"

![Give others access](/img/give_others_access.png)

* In the pop-up that appears click on the button titled "Add..."

![User account pop-up](/img/user_account_pop_up.png)

* Click the button titled "Browse..."

![Add user account](/img/add_user_acccount.png)

* On the select user screen click on the button titled "Locations..."

![Select user account](/img/select_user_acccount.png)

* Expand the tree and select your Active Directory domain. Then click "OK"

![Select location](/img/select_location.png)

* You will now be back on the "select user" screen. Click the button titled "Advanced..."

![Select user account](/img/select_user_acccount.png)

* Type the beginning of the name of the user account in the "Name" field and then click the button titled "Find Now". If successful you should see a list of matching user accounts in the "Search results" section. Click on the one you want and then click on the "OK" button

![Search for user account](/img/search_user_acccount.png)

* You will now be back on the "select user" screen, click the "OK" button

![Select user account filled](/img/select_user_acccount_filled.png)

* You will now be back at the original add user pop-up but the information will be filled in. Click the button titled "Next"

![Add user account filled](/img/add_user_acccount_filled.png)

* Select "Administrator" from the list and click "Next"

![Select admin](/img/select_admin.png)

* Click "Finish" on this last screen and you should be good to go.

The next time the user logs in they will have administrator rights to the computer