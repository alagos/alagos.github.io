---
layout: post
title: Usando tablet Android como pantalla extendida en Ubuntu
thumbnail: tablet
author: Alter
tags:
 - android
 - ubuntu
---

Buscando alguna utilidad extra que darle a mi tablet, pensé en ocuparlo como un escritorio extendido y así mostrar más información que, para un desarrollador, nunca es suficiente.

Vale decir que estas instrucciones deberían funcionar para cualquier versión de Ubuntu / dispositivo Android, pero solo para dejar constancia, parte de mi stack es:

* Tablet con Android Lolipop (5.0.1)
* Xubuntu Trusty (14.04)
* Drivers propietarios de Nvidia (340.46)

##Configuración Ubuntu

Primero que nada, instalar los programas necesarios para que el tablet pueda conectarse:

{% highlight bash %}
sudo apt-get install xserver-xorg-video-dummy x11vnc -y
{% endhighlight %}

* xserver-xorg-video-dummy: Permitirá configurar lo que se muestre en el tablet como un monitor virtual de la sesión de X actual.
* x11vnc: Servidor VNC que compartirá la sesión X a cualquier cliente VNC.

Luego se debe editar el archivo `/etc/X11/xorg.conf` y en la sección `ServerLayout` agregar las siguientes líneas:

        Screen      1  "Screen1" RightOf "Screen0"
        Option         "Xinerama" "1"

Donde `RightOf` indica dónde nuestro monitor virtual se posicionará, en este caso, a la derecha del monitor principal.
Otras opciones son `LeftOf`, `Above` o `Below`.
Para opciones más avanzadas de posicionamiento, [revisar la documentación de xorg.conf][1]

En mi caso, la sección `ServerLayout` queda como sigue:

    Section "ServerLayout"
        Identifier     "Layout0"
        Screen      0  "Screen0" 0 0
        Screen      1  "Screen1" RightOf "Screen0"
        InputDevice    "Keyboard0" "CoreKeyboard"
        InputDevice    "Mouse0" "CorePointer"
        Option         "Xinerama" "1"
    EndSection

También agregar al final del archivo la información relacionada al screen virtual creado:

    ############
    ## Xdummy ##
    ############

    Section "Device"
      Identifier       "Device1"
      Driver           "dummy"
      VideoRam          256000
    EndSection

    Section "Monitor"
      Identifier       "Monitor1"
    EndSection

    Section "Screen"
      Identifier       "Screen1"
      Device           "Device1"
      Monitor          "Monitor1"
      SubSection       "Display"
        Modes          "1536x2048"
      EndSubSection
    EndSection

    ################
    ## Fin Xdummy ##
    ################

A pesar de que no es necesario para que funcione correctamente, agregué `SubSection "Display"` en la configuración del Screen donde he seteado la resolución de mi tablet que es `1536x2048`.

Para que los cambios se reflejen, debemos reiniciar el X display manager:

{% highlight bash %}
sudo service lightdm restart
{% endhighlight %}

Finalmente correremos el servidor vnc:

{% highlight bash %}
x11vnc -noxdamage -clip xinerama1 -forever
{% endhighlight %}

La opción `-clip xinerama1` permitirá solamente compartir lo que se muestre en el display virtual creado.

## Configuración Android

Primero es necesario instalar un cliente VNC. El mas popular es [VNC Viewer][2], aunque personalmente me dio problemas con el cursor que se muestra siempre centrado en el tablet, así que terminé usando [VNC per Android][3]. De cualquier forma si ninguno de estos dos cumplen tus necesidades, hay [otros clientes](https://play.google.com/store/search?q=vnc+client&c=apps) que pueden probar a instalar.

Luego ejecutar la aplicación y configurar la conexión al equipo con Ubuntu. En mi caso:

![Conexión con VNC por Android]({{ "/images/Screenshot_2015-05-17-16-36-05.png" | prepend:site.baseurl }})



<cite>
  (Via: [Use any Tablet/Smartphone as extended monitor wireless in GNU/Linux][4] y [Xinerama, un ejemplo práctico][5])
</cite>

[1]: ftp://www.x.org/pub/X11R6.8.2/doc/xorg.conf.5.html#toc13
[2]: https://play.google.com/store/apps/details?id=com.realvnc.viewer.android
[3]: https://play.google.com/store/apps/details?id=android.androidVNC
[4]: http://slackfa.blogspot.com/2014/02/use-any-tabletsmartphone-as-extended.html
[5]: http://chernando.eu/doc/xinerama/