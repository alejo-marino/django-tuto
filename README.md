# tutorial-site + Language middleware proof of concept
This is a training repository for the [8-step django tutorial](https://docs.djangoproject.com/en/5.1/intro/tutorial01/)
To run the server make sure you're standing in the root folder of the repository and run (Look at the "Required Dependencies" before running):
```
python tutorial-site/manage.py runserver
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
python -m pip install polls/dist/polls-0.1.tar.gz
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
    output = _("Welcome to my site.")
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
    prefix_default_language=False   # for more info look at section 4 (Switching languages)
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
This is necessary since Django has a hard time finding and procesing locales outside of the BASE_DIR so we can circumvent that with a temporary symbolic link. There may be a workaround I haven't found.
```
ln -s ~/.../polls/choice_polls ./
```

#### Instruct makemessages to follow the symlink and specify the language to get translations for
```
./manage.py makemessages -s -l es
```

#### Create a folder called "translations" (name can be whatever you please, using translations for clarity) for your third oarty applications' translations
mkdir -p ./\<proj>/translations/\<app>  (exclude project if you're already sitting in the project's root (BASEDIR))

For our example:
```
mkdir -p ./polls_translation/choice_polls
```

#### Copy inside, the translation files from the package's locale directory to your project's translations directory
BASE_DIR / 'translations' / '\<app>' / \<your_lang>
For our example:
```
cp -R ./choice_polls/locale/es ./polls_translation/choice_polls/es
```

#### Result:
If \<your_lang> is "es", the resulting folder will be:
```
BASE_DIR / 'polls_translation' / 'choice_polls' / 'es',
```

#### Set accordingly the LOCALE_PATHS in settings.py to allow Django to find the translations:
```
LOCALE_PATHS = [
    ...,
    BASE_DIR / 'polls_translation' / 'choice_polls',
    ...,
]
```

#### Remove the symlink (make sure you're in the BASE_DIR of your app where you created it)
rm ./\<app>

For our example:
```
rm ./choice_polls
```

#### Translate the .po file (use vim, nano, or any other text editor):
vim \<proj>/translations/\<app>/\<your_lang>/LC_MESSAGES/django.po.
For our example:
```
vim tutorial-site/polls_translation/choice_polls/es/LC_MESSAGES/django.po
```

#### Compile messages
```
./manage.py compilemessages
```

At this point django will message that a .mo file was created inside the directory where the new .po file exists.

You should now be good to go and your app ready to start using the necessary translations!

#### 4. Switching languages

#### Switching through code
Let's now look at different methods of infering the language to be used. We already set the languages we want our app to be available in LANGUAGES. We'll use the get_language() function to obtain the language used for a certain thread, the program will infer this from settings.LANGUAGE_CODE. If we want to set a thread's language during execution, we should call the activate(language) function. An example:
```
from django.utils import translation

def welcome_translated(language):
    cur_language = translation.get_language()
    try:
        translation.activate(language)
        text = translation.gettext("welcome")
    finally:
        translation.activate(cur_language)
    return text
```

If we called welcome_translated() with the 'es' parameter and our default language was 'en', the function will return text with the value 'bienvenido', the function sets the language back to 'en' before finalizing the function so that the thread's language won't be altered. 
The override() function does the same thing but more concisely:
```
from django.utils import translation

def welcome_translated(language):
    with translation.override(language):
        return translation.gettext("welcome")
```

#### Django's way of inferring language

1. First, Django will look for a language prefix in a URL to try to infer it's language (we already did this back in Section ***2.*** when we set our urls to use i18n_patters()). This way, if we want our site to be displayed in English we can access www.mysite.com/en/home (being 'en' the default language, we should access it with www.mysite.com/home), if we're looking to have it display another supported language like Spanish we can access it with www.mysite.com/es/home.
2. Failing 1, it looks for a cookie (LANGUAGE_COOKIE_NAME), more details on it [here](https://docs.djangoproject.com/en/5.1/topics/i18n/translation/#language-cookie)
3. Failing 2, it looks at the Accept-Language HTTP header sent by the user's browser. Django tries each of the languages in this header until one matches with the supported LANGUAGES.
4. Failing all previous options, Django will default to using the LANGUAGE_CODE specified in settings.py

### Summary

If we wanna add a new language, let's say Chinese (cn) to our app, the steps we should follow should be:

1. Add the new language in the settings.py file for our main app:
```
LANGUAGES = [
    ...
    ('cn', _('Chinese')),
    ...
]
```
2. Head to the packages' directories we want to translate (in our case /polls)
3. Execute the following command to create the .po file
```
django-admin makemessages -l cn
```
4. Fill out the required translated strings
5. Run the following command to compile the .po file into an .mo file
```
./manage.py compilemessages
```
6. Head to our project's base directory (in our case /tutorial-site)
7. Execute the following command to create the .po file (this one's for the files in your app, not the package/s):
```
./manage.py makemessages -l cn
```
8. Create the symbolic link to the required package that we're looking to also translate:
```
ln -s ~/.../polls/choice_polls ./
```
9. Copy the contents related to translation files:
```
cp -R ./choice_polls/locale/cn ./polls_translation/choice_polls/cn
```
10. Compile all translation files:
```
./manage.py compilemessages
```
11. Remove the symlink and run
```
rm ./choice_polls
./manage.py runserver
```

### Useful links to further undestand:
- Everything related to the translation module: https://docs.djangoproject.com/en/5.1/topics/i18n/translation/
- For more information on how Django gets what language it should display, visit: https://docs.djangoproject.com/en/5.1/topics/i18n/translation/#how-django-discovers-language-preference
- https://www.marinamele.com/taskbuster-django-tutorial/internationalization-localization-languages-time-zones/
- https://automationpanda.com/2018/04/21/django-admin-translations/

### Español: Guía paso-a-paso configurar el Middleware de Traduccion de Django

Para agregar un cierto idioma a la traducción predeterminada de Django, eche un vistazo a LANGUAGES en tutorial-site/settings.py. Primero que nada, necesitamos hacer una distinción entre:

- Traducciones para el proyecto en cuestión
- Traducciones para los paquetes utilizados por el proyecto

Quiero llamar la atención sobre esto ya que esta guía se enfoca principalmente en el segundo tipo, ya que el primero es casi trivial y no requiere pasos adicionales. Asumiré que los paquetes requeridos por su aplicación están instalados en un entorno virtual o [ virtual-env](https://docs.python.org/3/library/venv.html), también asumiré que está utilizando un sistema operativo basado en UNIX, o al menos ejecutando algo como WSL para los comandos que usaré.

#### 1. ¿Cómo elijo qué traducir?
Bastante simple, digamos que tengo la siguiente vista/view como la página de inicio de mi sitio:

```
from django.http import HttpResponse


def my_view(request):
    output = ("Bienvenido a mi sitio.")
    return HttpResponse(output)
```

Todo lo que tengo que hacer es identificar el texto que me gustaría traducir (la traducción no solo funciona en cadenas estáticas, sino también en valores calculados o variables) e importar gettext_lazy() (la mayoría de las veces debería usar esto en lugar de gettext()) para hacer la traducción, es una práctica común importar esta función con el alias '_' pero, por supuesto, no es obligatorio:
```
from django.http import HttpResponse
from django.utils.translation import gettext_lazy as _


def my_view(request):
    output = _("Bienvenido a mi sitio.")
    return HttpResponse(output)
```

Lo has hecho! Ahora la cadena "Bienvenido a mi sitio." ha sido marcada para traducción, pero no termina ahí.

Veamos un ejemplo de utilizacion mas avanzado:

```
def my_view(request, m, d):
    output = _("Today is %(month)s %(day)s.") % {"month": m, "day": d}
    return HttpResponse(output)
```

Ahora digamos que quiero dejar un comentario para la persona que tenga que traducir la palabra que marque, todo lo que tenemos que hacer es:
```
def my_view(request):
    # Translators: Este mensaje solo aparece en la pagina de inicio!
    output = _("Bienvenido a mi sitio.")
```
Esto permitira que cualquier persona que la persona que traduzca esta string particular, pueda ver el mensaje: 'Este mensaje solo aparece en la pagina de inicio!'.

Si estas intentando traducir palabras pluralidazas, [esto](https://docs.djangoproject.com/en/5.1/topics/i18n/translation/#pluralization) deberia ser facil de entender y replicar ahora.

Ahora que hemos visto cómo marcar ciertas palabras para ser traducidas, pasemos a generar dichas traducciones (ciertos componentes como la página de administración predeterminada de Django vienen con sus traducciones ya establecidas que se pueden usar simplemente cambiando el idioma de la aplicación).


#### 2. Habilitando la traducción en mi aplicación
Podemos comenzar creando una carpeta 'locale' en el directorio principal de su aplicación (BASE_DIR)

```
mdkir <tu_app>/locale
```

Luego, diríjase a settings.py y asegúrese de tener las siguientes configuraciones:
```
LANGUAGE_CODE = 'en-ar'  # este debería ser su idioma predeterminado deseado
USE_I18N = True
USE_L10N = True
```

Agregue el directorio 'locale' correspondiente que acaba de crear a LOCALE_PATHS en settings.py:
```
BASE_DIR = Path(__file__).resolve().parent.parent

LOCALE_PATHS = [
    BASE_DIR / 'locale',
]
```

Agregue el middleware de Django en settings.py para que podamos acceder a su funcionalidad de traducción:
```
MIDDLEWARE = [
    # ...
    'django.middleware.locale.LocaleMiddleware',
    # ...
]
```

Mirando su archivo urls.py en su aplicación principal, debe usar la función i18n_patterns() para agregar internacionalización:
```
from django.conf.urls.i18n import i18n_patterns
from django.contrib import admin
from django.urls import path
 
urlpatterns = i18n_patterns(
    # ...
    path('admin/', admin.site.urls),
    # ...
 
    # Si no se da un prefijo, use el idioma predeterminado
    prefix_default_language=False   # para mas informacion mire la Sección 4 (Cambiando lenguajes)
)
```

Diríjase a settings.py y agregue cualquier idioma (con su código de idioma asociado) que desee que su aplicación soporte en el array LANGUAGES (vea una lista de idiomas y sus códigos [aquí](https://www.w3schools.com/tags/ref_language_codes.asp)):
```
from django.utils.translation import gettext_lazy as _
 
LANGUAGES = [
    ('es-ar', _('Español (Argentina)')),
    ('en', _('Inglés')),
]
```

#### 3. Generando y traduciendo las traducciones requeridas
#### Desde la raíz de su proyecto (BASE_DIR). Cree un enlace simbólico o symlink a la aplicación que necesita ser traducida (en nuestro caso, choice_polls)
Esto es necesario ya que Django tiene dificultades para encontrar y procesar locales fuera del BASE_DIR, por lo que podemos sortear eso con un enlace simbólico temporal. Puede haber una solución alternativa que no haya encontrado.
```
ln -s ~/.../polls/choice_polls ./
```

#### Instruya a makemessages para que siga el enlace simbólico y especifique el idioma para obtener traducciones
```
./manage.py makemessages -s -l en
```

#### Cree una carpeta llamada "traducciones" (el nombre puede ser el que desee, uso traducciones para mayor claridad) para las traducciones de sus aplicaciones de terceros
mkdir -p ./\<proj>/traducciones/\<app>  (excluya el proyecto si ya está en la raíz del proyecto (BASE_DIR))

Para nuestro ejemplo:
```
mkdir -p ./polls_translation/choice_polls
```

#### Copie dentro, los archivos de traducción del directorio locale del paquete al directorio de traducciones de su proyecto

BASE_DIR / 'traducciones' / '\<app>' / \<your_lang>

For our example:
```
cp -R ./choice_polls/locale/en ./polls_translation/choice_polls/en
```

#### Resultado:
Si \<your_lang> es "en", la carpeta resultante será:
```
BASE_DIR / 'polls_translation' / 'choice_polls' / 'en',
```

#### Configure adecuadamente los LOCALE_PATHS en settings.py para permitir que Django encuentre las traducciones:
```
LOCALE_PATHS = [
    ...,
    BASE_DIR / 'polls_translation' / 'choice_polls',
    ...,
]
```

#### Elimine el enlace simbólico (asegúrese de estar en el BASE_DIR de su aplicación donde lo creó)
rm ./\<app>

Para nuestro ejemplo:
```
rm ./choice_polls
```

#### Traduzca el archivo .po (use vim, nano u otro editor de texto):

vim \<proj>/translations/\<app>/\<your_lang>/LC_MESSAGES/django.po.

Para nuestro ejemplo:
```
vim tutorial-site/polls_translation/choice_polls/en/LC_MESSAGES/django.po
```

#### Compile las traducciones
```
./manage.py compilemessages
```

En este punto, Django indicará que se creó un archivo .mo dentro del directorio donde existe el nuevo archivo .po.

¡Ahora debería estar listo y su aplicación lista para comenzar a usar las traducciones necesarias!

#### 4. Cambiando idiomas

#### Cambiando a través del código

Veamos ahora diferentes métodos para inferir el idioma a utilizar. Ya configuramos los idiomas en los que queremos que nuestra aplicación esté disponible en LANGUAGES. Usaremos la función get_language() para obtener el idioma utilizado para un cierto hilo, el programa inferirá esto de settings.LANGUAGE_CODE. Si queremos establecer el idioma de un hilo durante la ejecución, debemos llamar a la función activate(lenguaje). Un ejemplo:
```
from django.utils import translation

def welcome_translated(language):
    cur_language = translation.get_language()
    try:
        translation.activate(language)
        text = translation.gettext("bienvenido")
    finally:
        translation.activate(cur_language)
    return text
```

Si llamamos a welcome_translated() con el parámetro 'en' y nuestro idioma predeterminado era 'es_AR', la función devolverá el texto con el valor 'welcome', la función restablece el idioma a 'es_AR' antes de finalizar la función para que el idioma del hilo no se altere. La función override() hace lo mismo pero de manera más concisa:
```
from django.utils import translation

def welcome_translated(language):
    with translation.override(language):
        return translation.gettext("bienvenido")
```

#### La forma en que Django infiere el idioma

1. Primero, Django buscará un prefijo de idioma en una URL para intentar inferir su idioma (ya hicimos esto en la Sección 2. cuando configuramos nuestras urls para usar i18n_patterns()). De esta manera, si queremos que nuestro sitio se muestre en inglés, podemos acceder a www.mysite.com/en/home, si queremos que se muestre en otro idioma compatible como español, podemos acceder a él con www.mysite.com/es_AR/home (al ser el idioma default el es_AR, deberiamos acceder con www.mysite.com/home).
2. Si falla el punto 1, busca una cookie (LANGUAGE_COOKIE_NAME), más detalles sobre esto [aqui](https://docs.djangoproject.com/en/5.1/topics/i18n/translation/#language-cookie)
3. FSi falla el punto 2, busca en el encabezado HTTP Accept-Language enviado por el navegador del usuario. Django prueba cada uno de los idiomas en este encabezado hasta que uno coincide con los idiomas compatibles.
4. Si fallan todas las opciones anteriores, Django usará el LANGUAGE_CODE especificado en settings.py

### Resumen

Si queremos agregar un nuevo idioma, digamos Chino (cn) a nuestra aplicación, los pasos que debemos seguir son:

1. Agregar el nuevo idioma en el archivo settings.py de nuestra aplicación principal:
```
LANGUAGES = [
    ...
    ('cn', _('Chino')),
    ...
]
```
2. Diríjase a los directorios de los paquetes que queremos traducir (en nuestro caso /polls)
3. Ejecute el siguiente comando para crear el archivo .po
```
django-admin makemessages -l cn
```
4. Complete el archivo .po con las strings de traduccion
5. Ejecute el siguiente comando para compilar el archivo .po en un archivo .mo
```
./manage.py compilemessages
```
6. Diríjase al directorio base de nuestro proyecto (en nuestro caso /tutorial-site)
7. Ejecute el siguiente comando para crear el archivo .po (este es para los archivos en su aplicación, no los paquetes):
```
./manage.py makemessages -l cn
```
8. Cree el enlace simbólico al paquete requerido que también estamos buscando traducir:
```
ln -s ~/.../polls/choice_polls ./
```
9. Copie los contenidos relacionados con los archivos de traducción:
```
cp -R ./choice_polls/locale/cn ./polls_translation/choice_polls/cn
```
10. Compile todos los archivos de traducción:
```
./manage.py compilemessages
```
11. Elimine el enlace simbólico y ejecute el servidor
```
rm ./choice_polls
./manage.py runserver
```

### Enlaces útiles para entender mejor:
- Todo lo relacionado con el módulo de traducción: https://docs.djangoproject.com/en/5.1/topics/i18n/translation/
- Para obtener más información sobre cómo Django obtiene el idioma que debe mostrar, visite: https://docs.djangoproject.com/en/5.1/topics/i18n/translation/#how-django-discovers-language-preference
- https://www.marinamele.com/taskbuster-django-tutorial/internationalization-localization-languages-time-zones/
- https://automationpanda.com/2018/04/21/django-admin-translations/
