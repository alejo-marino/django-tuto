# tutorial-site + Language middleware proof of concept
This is a training repository for the [8-step django tutorial](https://docs.djangoproject.com/en/5.1/intro/tutorial01/)
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

## Translation

Translation for the admin page was added using [Django's Translation Middleware](https://docs.djangoproject.com/en/5.1/topics/i18n/translation/)

### English: Step-by-Step Guide for setting up Django's Translation Middleware

To add a certain language to django's default translation, take look at LANGUAGES in tutorial-site/setting.py. First off, we need need to make a distinction between:

- Translations for the desired proyect
- Translations for packages used in the desired proyect

I want to draw attention to this since this guide focuses mostly on the second type, since the first is almost trivial and requires no extra steps. I'll assume that your app's required packages are installed on a [virtual-environment](https://docs.python.org/3/library/venv.html), I'll also assume that you're running a UNIX-based OS, or at least running something such a WSL for the commands I'll use.

#### 1. How do I choose what to translate?
Quite simple, let's say I have the following view as my site's landing page:
```
from django.http import HttpResponse


def my_view(request):
    output = ("Welcome to my site.")
    return HttpResponse(output)
```
All I have to do is identify the text I'd like to translate (translation not only works on static strings, but also on computed valueis or variables) and import gettext_lazy() (most of the times you should be using this instead of gettext()) to do the translation, it's common practice to import this function with the '_' alias but of course it's not mandatory:
```
from django.http import HttpResponse
from django.utils.translation import gettext_lazy as _


def my_view(request):
    output = _("Welcome to my site.")
    return HttpResponse(output)
```
You've done it, now the string "Welcome to my site." has been marked for translation, but it doesn't end there.

Let's show a more advanced example of utilization:

```
def my_view(request, m, d):
    output = _("Today is %(month)s %(day)s.") % {"month": m, "day": d}
    return HttpResponse(output)
```

Now let's say I want to leave a comment for whoever will translate this, all we have to do is the following:
```
def my_view(request):
    # Translators: This message appears on the home page only
    output = gettext("Welcome to my site.")
```
This will let whoever's translating this particular string to see the 'This message appears on the home page only' message.

If you're trying to translate pluralized words, [this](https://docs.djangoproject.com/en/5.1/topics/i18n/translation/#pluralization) should be easy to understand and replicate now

Now that we've seen how to mark certain words for translation, let's move on to generating said translations (Certain components like Django's default admin page come with their already established translations that can be used just by switching the app's language).

#### 2. Enabling translation in my app
We can start by creating a 'locale' folder in your apps's main directory (BASE_DIR)
```
mdkir <your_app>/locale
```

Then, head over to settings.py and make sure you have the following settings:
```
LANGUAGE_CODE = 'en-us'  # this should be your desired default language
USE_I18N = True
USE_L10N = True
```

Add the corresponding 'locale' directory you just created to LOCALE_PATHS in settings.py:
```
BASE_DIR = Path(__file__).resolve().parent.parent

LOCALE_PATHS = [
    BASE_DIR / 'locale',
]
```

Add Django's middleware in settings.py so we can access it's translation functionality:
```
MIDDLEWARE = [
    # ...
    'django.middleware.locale.LocaleMiddleware',
    # ...
]
```

Looking at your urls.py file in your main app, you should use the i18n_patters() function to add internationalization:
```
from django.conf.urls.i18n import i18n_patterns
from django.contrib import admin
from django.urls import path
 
urlpatterns = i18n_patterns(
    # ...
    path('admin/', admin.site.urls),
    # ...
 
    # If no prefix is given, use the default language
    prefix_default_language=False
)
```

Head over to settings.py and add any language (with it's associated language code) you want your app to support in the LANGUAJES array (see a list of languages and their codes [here](https://www.w3schools.com/tags/ref_language_codes.asp)):
```
from django.utils.translation import gettext_lazy as _
 
LANGUAGES = [
    ('en', _('English')),
    ('es', _('Spanish')),
]
```

#### 3. Generating and translating the required translations
#### From the root of your project (BASE_DIR). Create a symlink to the application that needs to be translated (in our case, choice_polls)
```
ln -s ~/.../polls/choice_polls ./
```

#### Instruct makemessages to follow the symlink
./manage.py makemessages -s  

#### Create a folder called "translations" (name can be whatever you please, using translations for clarity) for your third oarty applications' translations
mkdir ./\<proj>/translations/\<app>

For our example:
```
mkdir ./tutorial-site/polls_translation/choice_polls
```

#### Copy inside, the translation files from the package's locale directory to your project's translations directory
BASE_DIR / 'translations' / '\<app>' / \<your_lang>
Four our example:
```
cp -R ./choice_polls/locale/es ./tutorial-site/polls_translation/choice_polls
```

#### Result:
If \<your_lang> is "es", the resulting folder will be:
```
BASE_DIR / 'polls_translation' / 'choice_polls' / 'es',
```

#### Set accordingly the LOCALE_PATHS in settings.py to allow Django to find the translations:
...
LOCALE_PATHS = [
    ...,
    BASE_DIR / 'polls_translation' / 'choice_polls' / 'es',
    ...,
]
...

#### Remove the symlink (make sure you're in the BASE_DIR of your app where you created it)
rm ./\<app>

For our example:
```
rm ./choice-polls
```

#### Translate the .po file (use vim, nano, or any other text editor):
vim \<proj>/translations/\<app>/\<your_lang>/LC_MESSAGES/django.po.
For our example:
```
vim tutorial-site/polls_translation/choice-polls/en/LC_MESSAGES/django.po
```

#### Compile messages
```
./manage.py compilemessages
```

At this point django will message that a .mo file was created inside the directory where the new .po file exists.




#### Useful links to further undestand:
- https://www.marinamele.com/taskbuster-django-tutorial/internationalization-localization-languages-time-zones/
- https://automationpanda.com/2018/04/21/django-admin-translations/
- For more information on how Django gets what language it should display, visit: https://docs.djangoproject.com/en/5.1/topics/i18n/translation/#how-django-discovers-language-preference
