# tutorial-site + Language middleware proof of concept
This is a training repository for the 8-step django tutorial found at https://docs.djangoproject.com/en/5.1/intro/tutorial01/
To run the server make sure you're standing in the root folder of the repository and run (Look at the "Required Dependencies" before running):
```
python django-tutorial/manage.py runserver
```

## Required Dependencies

To be able to run this program, position yourself in the polls directory and run:
```
python -m build
```
Then position yourself at the root of the program (you should see both tutorial-site and polls) and run the following commands
```
sudo apt-get install gettext

python -m pip install Django
python -m pip install django-debug-toolbar
python -m pip install polls/dist/choice_polls-0.1.tar.gz
```


### Translation

Translation for the admin page was added using Django's middleware, for more information on how Django gets what language it should display, visit: https://docs.djangoproject.com/en/5.1/topics/i18n/translation/#how-django-discovers-language-preference
To add a certain language to django's default translation, take look at LANGUAGES in tutorial-site/setting.py

To generate the .po (translation files) for a language X (we'll use 'es' in this example), run:
```
django-admin makemessages -l es
django-admin compilemessages
```

Useful links to further undestand:
- https://www.marinamele.com/taskbuster-django-tutorial/internationalization-localization-languages-time-zones/
- https://automationpanda.com/2018/04/21/django-admin-translations/
