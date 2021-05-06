​                                                                                              





​                                                                                                     <center> ![image-20210506104427925](https://tva1.sinaimg.cn/large/008i3skNgy1gq8sv4q7cqj303k03kweo.jpg) </center>          



### ¿Que es este repositorio?

Este proyecto es la implementación de una web creada en notion.so convertida a una web estatica mediante GitHub Actions y un script en python llamado **[loconotion](https://github.com/leoncvlt/loconotion )** 

### ¿Pero por qué?

> **[Notion](https://notion.so)** permite crear notas y documentos, wikis, gestionar proyectos y tareas y, en realidad, hacer casi cualquier cosa. ... **Notion** es accesible desde la web, desde las apps para Windows y Mac y desde las respectivas para iOS y Android.

[Notion.so](https://notion.so) es una aplicación genial, pero no está orientado a la creación páginas web porque tiene algunas carencias, como por ejemplo que para ser usado como página web es bástante lento, no dispone de soporte para SEO ni tampoco la personalización con dominios propios, pero sin embargo no he querido desaprovechar el potencial de su editor.

### Motivación

Había estado probando algunas soluciones parecidas, Pero o requerían de servidor o eran de pago. Me puse a buscar en GitHub y descubrí **loconotion** vi que lo que hacia era generar contenido estatico a partir de una URL de notion.so y se me encendio la bombilla. **¿Porque no usar  GitHub Pages?** 

Podía crear la página estática y subirla directamente, pero me encanta complicarme la vida, así que empece a darle vida a mi idea. Sí puedo generar  contenido estático y además puedo ejecutar código en contenedores, **¡Pues ya está!** Puedo crear un pipeline para desplegar directamente la web en GH Pages y puedo aprovechar la cache de CloudFlare para **reducir el TTFB** conseguir una velocidad de cargá mucho más rápida, y este es el resultado.

![image-20210502220617140](https://tva1.sinaimg.cn/large/008i3skNgy1gq4q3ceqlwj313j0hvgqm.jpg)



**Además**

* Proyecto Serverless
* Control de metatags
* Ejecución de JavaScript Adicional
* Favicon personalizado
* CSS personalizado
* Purgado de cache en el deploy.
* Práctico como crear una Pipeline.

### ¿Como?

Si quieres desplegar tú propia web básada en notion tan solo tienes que clonar este repositorio.

```
$ git clone https://github.com/aitorroma/aitorroma
$ cd aitorroma
$ rm -Rf .git
$ git init
$ git add .
$ git commit -m 'First commit'
$ git remote add origin <remote repository URL>
$ git remote -v
$ git push origin master
```

Puedes editar el fichero _**project/[aitorroma.com.toml](https://github.com/aitorroma/aitorroma/blob/main/project/aitorroma.com.toml)**_

Editar el valor:

```
page =
```

Por tú página creada en notion.so

También debes editar los metatags del sitio:

```
[[site.meta]]
  name = "title"
  content = "Aitor Roma, Página Oficial"
  [[site.meta]]
  name = "description"
  content = "Página personal de Aitor Roma, Apasionado del Software Libre, Cloud Computing y Growth Hacking"
```

Por último puedes modificar el contenido de:

```
project
    ├── aitorroma.com.toml
    ├── assets
    │   ├── button.html
    │   ├── calendar.html
    │   ├── coffe.html
    │   └── player.html
    ├── custom-script.js
    └── favicon-16x16.png
```

Básicamente los ficheros:

* custom-script.js
* favicon-16x16.png

**custom-scripts.js** te permite añadir contenido en javascripts en mi caso he añadido las estadísticas de Matomo. 

**favicon-16x16.png** es el favicon de la web



Los ficheros html de *assets* los cuelgo para poder cargarlos como contenido embebido en la web y añadir algunas cosas que no soporta de facto notion.so pero que se pueden crear fácimente usando html5 y css. 

### Configuración de los Secrets

Para que funcione el deploy se debe configurar los siguientes secrets.

![Captura de pantalla 2021-05-02 a las 20.51.11](https://tva1.sinaimg.cn/large/008i3skNgy1gq4q3see70j30ol07pmxp.jpg)

Los secrets vamos a categorizarlos en dos tipos los secrets de **CloudFlare** para limpiar la cache y el secret de **Github Pages** que permitira subir el contenido estático generado a la rama ***gh-pages***



### Creación de los secrets de CloudFlare

Partimos de que ya tenemos un dominio en [Cloudflare](https://cloudflare.com), vamos a necesitar generar un token de la API de usuario que nos permita gestionar la cache para ello iremos al apartado [Tokens de API](https://dash.cloudflare.com/profile/api-tokens) de nuestro Perfil.

Una vez allí le daremos a **Crear Token** 

Crearemos un **Token personalizado** algo parecido a esto pero seleccionando nuestro dominio.

![Captura de pantalla 2021-05-02 a las 21.16.11](https://tva1.sinaimg.cn/large/008i3skNgy1gq4q4jd6qgj30vm0llgnm.jpg)



Una vez configurada esta pantalla nos aparecerá un  Token formado por  números y letras que debemos guardar, porque ahora lo usaremos.



Ahora debemos ir a GitHub en nuestro repositorio y hacer click en **[⚙️](https://emojiterra.com/es/engranaje/)Settings** / **Secrets**

Allí le damos a **New repository secret**

En está pantalla configuraremos el campo **Name** con el valor **CLOUDFLARE_TOKEN** (*así en mayúsculas tal como está escrito.*)

En **Value** pegaremos la **token** que hemos **generado en CloudFlare**



![Captura de pantalla 2021-05-02 a las 21.22.29](https://tva1.sinaimg.cn/large/008i3skNgy1gq4q4wlfgtj30o30cp74m.jpg)

**Vamos a CloudFlare** en nuestro dominio y le damos a este icono.

![Captura de pantalla 2021-05-02 a las 21.26.27](https://tva1.sinaimg.cn/large/008i3skNgy1gq4q545hdlj302h036a9v.jpg)

Ahora en la parte inferior derecha de la pantalla aparecerá algo como esto.

![Captura de pantalla 2021-05-02 a las 21.27.55](https://tva1.sinaimg.cn/large/008i3skNgy1gq4q5blsylj30bz0aoq38.jpg) 

Copiaremos la **Id. de zona** y volveremos a las **Secrets de Github** crearemos una nueva secret **CLOUDFLARE_ZONE**:



**Nombre:** CLOUDFLARE_ZONE

**Value:** ( Id. de zona que hemos copiado)



### Configuración del Secret de GitHub

Ahora es el momento de crear una secret de GitHub que nos permitirá subir el repositorio en una nueva rama de que usaremos para servirla en **Github Pages**



Para eso vamos a nuestro perfil / Settings

![Captura de pantalla 2021-05-02 a las 21.34.01](https://tva1.sinaimg.cn/large/008i3skNgy1gq4q5k3691j305y0cw3yz.jpg)

Una vez dentro de settings vamos a **Developer Settings** / **Personal access tokens**, configuramos el repositorio así.

![Captura de pantalla 2021-05-02 a las 21.36.49](https://tva1.sinaimg.cn/large/008i3skNgy1gq4q5tlkujj30u80czdht.jpg)

Le damos abajo del todo en **Generate token** y volvemos al repositorio dentro de **Secrets**

Y configuramos una nueva secret

**Name:** GH_SECRET

**Value:** ( *Valor que hemos copiado al generar el token*)



### Configuración de GitHub Pages

Una vez tenemos todo esto configurado vamos a hacer que el script de deploy nos genere la web y vamos a configurar para poder servirla.

En nuestro repositorio vamos a la pestaña **Actions** y lanzamos una ejecución manual.

![Captura de pantalla 2021-05-02 a las 21.43.30](https://tva1.sinaimg.cn/large/008i3skNgy1gq4q614behj310m0c8abt.jpg)

Al terminar la ejecución nos habrá generado una nueva rama con el contenido estatico de nuestra web, que pasaremos a configurar en el apartado

 **[⚙️](https://emojiterra.com/es/engranaje/)Settings** / **Pages**

![image-20210502220913001](https://tva1.sinaimg.cn/large/008i3skNgy1gq4q6cj03hj30y40iy41q.jpg)

Dejaremos configurado:

#### Source

**Branch:** gh-pages 

**Path:** /root

#### Custom Domain

Definiremos nuestro dominio.



### Configuración DNS en CloudFlare

Entramos en nuestro dominio de CloudFlare y configuramos 4 registros de tipo **A** con las siguientes **IP**.

* 185.199.108.153
* 185.199.109.153
* 185.199.110.153
* 185.199.111.153

Así con la nube naranja activa.

![image-20210502220925953](https://tva1.sinaimg.cn/large/008i3skNgy1gq4q6kme6gj30tc091wff.jpg)

Una vez añadidas todas veremos esto pero con nuestro dominio.

![image-20210502220938003](https://tva1.sinaimg.cn/large/008i3skNgy1gq4q6s1x75j30wm0bo0u8.jpg)



### Activación de One Page Cache



Para activar una cache super efectiva de la web y que vaya a la velocidad del rayo iremos en el menu superior **Reglas** y crearemos una regla así.

![image-20210502220950131](https://tva1.sinaimg.cn/large/008i3skNgy1gq4q6zluksj30oq0dsgmy.jpg)

Debeis cambiar vuestro dominio y darle a **Guardar e implementar**



Si haceis algún cambio en vuestra página de notion.so debéis ir a Github en Actions y hacer un deploy manual, de lo contrario se realizara diariamente a las 0:00 UTC.

---------------------------------------------------------------------------------------------------------------------------------------------

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/J3J64AN17)

