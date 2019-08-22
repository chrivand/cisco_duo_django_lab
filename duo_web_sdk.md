# Demonstration of a Django site with Cisco Duo secondary authentication.

## Learning Objectives

* You will learn how to use the Django "example site" to create a simple web server with a login page. 
* You will learn how to add users and different duo secondary authentications per user.
* You will learn how to use the Duo WebSDK as secondary authentication.
* This makes it easy to get started with the WebSDK, as this is a ready to go example.

## Notes and dependencies

* Tested with Django 1.10 up until Django 2.0.2 on Python 2.7, Python 3.5 and 3.6. 
* Make sure that you Django version is 2.0.2, or if you prefer to use a higher version, then you need to be aware that the Login View has changed, and therefore some code alterations are needed. More info here: https://stackoverflow.com/questions/47801453/getting-this-error-in-django-login-missing-1-required-positional-argument-u
* Note that this example serves static files with the Django development server, which is not recommended for production.
* The examples herein do not add Duo "everywhere" nor the Django admin site. This is critical to recognize, because if you do not have 2FA on `/admin`, and you have `/admin` enabled, you essentially degrade to 1FA on the most sensitive part of your application.


## Installation instructions

1. Do a Git Clone of the following repository: https://github.com/duosecurity/duo_python
2. Change directory to the cloned folder, e.g.: *cd duo_python*
3. Create a Virtual Environement (venv) for all of the Python dependecies, e.g.: *python3.6 -m venv DUO_VENV* 
4. Activate your Virtual Environement: *source DUO_VENV/bin/activate*
5. Install all the requirements: *pip install -r requirements.txt* [Please take note that you either install Django 2.0.2, or modify the code!]
6. Add the Duo Integration Key, Secret Key and the API Host to *settings.py*. Please follow these steps to do so: https://duo.com/docs/duoweb
7. You are missing one value (the *DUO_AKEY*), which you have to generate yourself and keep secret from Duo. The security of your Duo application is tied to the security of your skey and akey. Treat these pieces of data like a password. They should be stored in a secure manner with limited access, whether that is in a database, a file on disk, or another storage mechanism. Always transfer them via secure channels, and do not send them over unencrypted email, enter them into chat channels, or include them in other communications with Duo.
8. Now you will actually generate an akey, which needs to be at least 40 characters long. You can generate a random string in Python by running these two commands in your Venv:

```
    python
    >>> import os, hashlib
    >>> print(hashlib.sha1(os.urandom(32)).hexdigest())
    >>> [generated Akey will be printed here]
    >>> exit()
```
9. Fill in the generated Akey in the *settings.py* file.
10. Now we will set up Django. First we need to run the initial database migration, by running these two commands in your Venv:

```
    python manage.py makemigrations
    python manage.py migrate
```

11. Now you can create a user, that will authenticate in your Django web app. Please run the following commands in your Venv: 

```
    python
    >>> import os
    >>> os.environ.setdefault("DJANGO_SETTINGS_MODULE", "example_site.settings")
    >>> import django
    >>> django.setup()
    >>> from django.contrib.auth.models import User
    >>> user = User.objects.create_user('john', 'lennon@thebeatles.com', 'johnpassword')
    >>> exit()
```

12. The name, email and password on the last you can obviously be changed to your liking, and you can also add more users by running the last line multiple times (with different names) before exiting the Python shell. 

13. Now you are ready to start the Django Web App. Please run the following commands in your Venv: 

```
    python manage.py runserver
```

14. You should see an output like this and by clicking on the link you will open up your Django Web App on the localhost:

```
    System check identified no issues (0 silenced).
    July 24, 2019 - 09:42:06
    Django version 2.0.2, using settings 'example_site.settings'
    Starting development server at http://127.0.0.1:8000/
    Quit the server with CONTROL-C.
    Performing system checks...
```

15. As mentioned, after starting the server, you can browse to `http://127.0.0.1:8000`, as shown below:

<img src="https://github.com/chrivand/cisco_duo_django_lab/blob/master/screenshots/terminal.png" width="550" height="300">

16. The username and password is what you have configured in step 11 and 12. The login will look something like this (small modifications have been made to the UI):

<img src="https://github.com/chrivand/cisco_duo_django_lab/blob/master/screenshots/primary_login.png" width="550" height="350">

17. After you login with the primary authentication (via the Django DB), you can click on *Continue to secondary auth content.* to advance to Duo secondary authentication. This will look something like this:

<img src="https://github.com/chrivand/cisco_duo_django_lab/blob/master/screenshots/primary_logged_in.png" width="550" height="350">

18. The first time that you continue to the secondary authentication, you will need to setup Duo just as usual (Mobile Phone + Push is recommended). You will then be prompted with the familiar Duo Prompt:

<img src="https://github.com/chrivand/cisco_duo_django_lab/blob/master/screenshots/secondary_auth.png" width="400" height="350">

19. When you have connected your iPhone with Duo Push, you will get prompted through the Duo Mobile App:

<img src="https://github.com/chrivand/cisco_duo_django_lab/blob/master/screenshots/phone_screenshot.png" width="200" height="400">

20. After doing the Duo secondary authentication, you will reach the Duo protected page. In *views.py* you can edit the *def duo_private(request)* Django HTML to your liking. Below is an example where a hyperlink is added that links to a file server:

<img src="https://github.com/chrivand/cisco_duo_django_lab/blob/master/screenshots/private_data_view.png" width="550" height="300">

21. On the Duo website you can create all kinds of policies, like blocking certain type of browsers or devices.
22. You can also logout and go back to the home menu, to login with a different Duo secondary authentication (e.g. YubiKey or TouchID).
23. Now you have successfully setup a Django Web App with Duo secondary authentication! See if you can change some aspects and make it more like a real life Web App. 

## Code Explanation

Now to find out where the actual calls are being made to the Duo WebSDK, we need to look at lines 88, 102, 105, 119 and 122 in the *duo_auth.py* file. Let's look at them one by one:

* Line 88: this is where the signature request is being done. As you can see it takes the Integration Key, the Secret Key and the generated Application key. Also, it takes the user from the primary authentication as well, in order for Duo to know which user is authenticating. Please have a look at the code snippet below:
```
    sig_request = duo_web.sign_request(
        settings.DUO_IKEY, settings.DUO_SKEY, settings.DUO_AKEY,
        duo_username(request.user)) 
```
* Line 102: this is where the user is prompted with the *duo_login.html* page. This contains the famous screen where you can decide which secondary authentication you will use. The page is rendered using the Django framework. Please have a look at the code snippet below:
```
    return render(request, 'duo_login.html', context)
```
* Line 105: on this line the response is being sent to Duo and it will be verified whether the user is allowed to continue. Please have a look at the code snippet below:
```
    duo_user = duo_web.verify_response(
        settings.DUO_IKEY, settings.DUO_SKEY, settings.DUO_AKEY,
        sig_response)
```
* Line 119: this is where a record is being made that the user is indeed verified and authenticated. Please have a look at the code snippet below:
```
    duo_authenticate(request)
```
* Line 122: on this line the next page will be rendered using the Django framework. Please have a look at the code snippet below:
```
    return HttpResponseRedirect(next_page)
```
