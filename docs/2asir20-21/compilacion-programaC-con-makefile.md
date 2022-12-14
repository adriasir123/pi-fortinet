---
title: "Práctica: Compilación de un programa en C utilizando un Makefile"
---

# Enunciado

1. Elige el programa escrito en C que prefieras y comprueba en las fuentes que exista un fichero Makefile para compilarlo

2. Realiza los pasos necesarios para compilarlo e instálalo en tu equipo en un directorio que no interfiera con tu sistema de paquetes (/opt, /usr/local, etc.)

3. La corrección se hará en clase y deberás ser capaz de explicar qué son todos los ficheros que se hayan instalado y realizar una desinstalación limpia.


# Desarrollo

## Primera parte

Elijo `gimp`, y dentro de su código fuente existe un fichero Makefile, efectivamente.  
De hecho son 2:

* Makefile.am
* Makefile.in

## Segunda parte

### **1. Requerimientos pre-compilación**

Resumen de paquetes requeridos junto con sus versiones
```
     Package Name         Version

+     ATK                  2.2.0
+     babl                 0.1.78
+     cairo                1.12.2
+     Fontconfig           2.12.4
+     freetype2            2.1.7
+     GDK-PixBuf           2.30.8
+     GEGL                 0.4.26
+     GIO
+     GLib                 2.56.2
+     glib-networking
+     GTK+                 2.24.32
+     HarfBuzz             0.9.19
+     libbzip2
+     libjpeg
+     liblzma              5.0.0
+     libmypaint           1.3.0
+     libpng               1.6.25
+     libpoppler-glib      0.50.0
+     librsvg              2.40.6
+     libtiff
+     Little CMS           2.8
+     mypaint-brushes-1.0
+     pangocairo           1.29.4
+     poppler-data         0.4.7
+     zlib
+     gexiv2               >= 0.10.6
```

Resumen para la instalación rápida de todos ellos
```
sudo apt update && sudo apt install poppler-data mypaint-brushes libpoppler-glib-dev liblzma5 libjpeg62-turbo glib-networking fontconfig libfreetype6 libglib2.0-0 gir1.2-glib-2.0 libcairo2 libgdk-pixbuf2.0-0 libharfbuzz-bin libpng16-16 lib64z1 libtiff-dev librsvg2-dev libpango1.0-0 liblcms2-2 libbz2-dev gtk+2.0 libatk1.0-0 libgexiv2-dev
```

**LIBRERÍA babl 0.1.78**  
Esa versión sólo se encuentra en unstable, así que...

* Añado la siguiente línea a `/etc/apt/sources.list`
     ```
     deb http://http.us.debian.org/debian unstable main non-free contrib
     ```
* Para evitar problemas, en el fichero `/etc/apt/preferences.d/priorities` añado el siguiente contenido
     ```
     Package: *
     Pin: release a=stable
     Pin-Priority: 700

     Package: *
     Pin: release a=unstable
     Pin-Priority: 600

     ```
     De esta manera, siempre que hagamos un `sudo apt install <nombre-paquete>`, por defecto lo hará de los que estén en rama `stable`, y no `unstable`. A no ser que le indiquemos lo contrario, claro. 
* Con todo esto hecho, ya SÍ podremos instalar la versión que necesitamos de babl
     ```
     sudo apt install -t unstable libbabl-dev
     ```
     Aunque antes de esto, tenemos que actualizar con la versión de unstable un paquete manualmente para que funcione
     ```
     sudo apt install -t unstable gcc-8-base
     ```

Con esto, ya tendríamos la librería a una versión que podemos usar
```
libbabl-dev:
  Installed: 1:0.1.82-1
  Candidate: 1:0.1.82-1
  Version table:
 *** 1:0.1.82-1 600
        600 http://http.us.debian.org/debian unstable/main amd64 Packages
        100 /var/lib/dpkg/status
     0.1.62-1 700
        700 http://deb.debian.org/debian buster/main amd64 Packages
```

**LIBRERÍA GEGL 0.4.26**  
Esa versión sólo se encuentra en unstable, así que...

*    ```
     sudo apt install -t unstable libgegl-dev
     ```
*    ```
     libgegl-dev:
     Installed: 1:0.4.26-1
     Candidate: 1:0.4.26-1
     Version table:
     *** 1:0.4.26-1 600
          600 http://http.us.debian.org/debian unstable/main amd64 Packages
          100 /var/lib/dpkg/status
          0.4.12-2 700
          700 http://deb.debian.org/debian buster/main amd64 Packages
     ```

**LIBRERÍA libmypaint**  
Por cierto warning a la hora de hacer el configure...
```
WARNING: libmypaint lower than version 1.4.0 is known to crash when
         parsing MyPaint 2 brushes. Please update.
```
...Mejor que tengamos la versión de unstable

```
sudo apt install -t unstable libmypaint-dev
```

### **2. Script configure**

> Sobre la opción `--prefix` a la hora de hacer un `./configure`
> https://stackoverflow.com/questions/62340836/what-does-prefix-in-configure-script-do-when-building-a-software-in-linux


En el propio directorio donde tenemos el código fuente, lo primero, es ejecutar `./configure`.  

¿Qué hace este script? Pues lo que nos dice el fichero `INSTALL` propio de gimp como manual de compilación es lo siguiente...

> The `configure' script examines your system, and adapts GIMP to run on
it.

Básicamente lo que esto quiere decir, es que se toman ciertos valores de nuestro sistema en específico, y partiendo de eso, se nos genera un "Makefile personalizado". Éste, posteriormente, será el usado para la compilación. 

Esto es lo que nos dice el propio script `configure`...

> Guess values for system-dependent variables and create Makefiles.

Una vez que ya tenemos una idea grosso modo de lo que hace este script, lo ejecutamos
```
./configure --disable-python --disable-gimp-console
```

> * Para prevenir ciertos errores que pueden aparecer     tras ejecutar este script...
>    ```
>    sudo apt install gcc build-essential intltool
>    ```
> * `--disable-python` = deshabilita la compilación de ciertas extensiones python que no necesitaríamos 
> * `--disable-gimp-console` = deshabilita la compilación del binario para la consola de gimp, que en principio tampoco es necesario

Después de hacer esto, tendríamos un output bastante extenso del script, del cual realmente nos interesa sólo la siguiente parte...

```
Building GIMP with prefix=/usr/local, datarootdir=${prefix}/share
Desktop files install into ${datarootdir}
```

`Prefix` es la opción que define dónde se moverán los binarios tras haberlos compilado y otros ficheros, cuando hagamos un `make install`. Esto pasará, claro está, después de haber hecho la compilación con `make`.

Con gimp, tenemos la suerte de que por defecto `/usr/local` será el prefix, pero habrá otros binarios que queramos comparar en los que no tengamos esa suerte. Simplemente si esto pasa, será cuestión de pasar un parámetro de `prefix` en el momento de hacer un `configure`, para indicar el directorio raíz que queramos.

Como último dato en este apartado, añadir que se nos ha generado nuestro "Makefile personalizado"

```
-rw-r--r--  1 vagrant vagrant    47922 Nov  3 05:08 Makefile
```

### **3. Compilación**

Ejecutamos `make`.  

Esto lo que hará es, teniendo en cuenta el Makefile que se generó, compilar todo lo necesario en los directorios dentro del código fuente.

Para un poco de información más detallada sobre esto, paso lo siguiente extraído del fichero `INSTALL`

```
The `make' command builds several things:
 - A bunch of public libraries in the directories starting with 'libgimp'.
 - The plug-in programs in the 'plug-ins' directory.
 - Some modules in the 'modules' subdirectory.
 - The main GIMP program 'gimp-2.10' in `app'.
```


### **4. Instalación**

Ejecutamos `sudo make install`.

> Necesitamos sudo porque estamos moviendo ficheros a `/usr/local`, directorio cuyo propietario es root, y otros usuarios no tienen permiso de escritura

Esto lo que hará es, mover todo lo compilado y generado al directorio raíz cuyo prefix hayamos establecido previamente.

Para más información, el fichero `INSTALL` dice lo siguiente...
```
By default, `make install' will install the package's files in
/usr/local/bin, /usr/local/lib, /usr/local/man, etc. 
```

Una vez que ya hemos instalado, muestro un resumen de `/usr/local` para que se vea la estructura que tenemos generada
```
/usr/local/
├── bin
├── etc
│   └── gimp
│       └── 2.0
├── games
├── include
│   └── gimp-2.0
│       ├── libgimp
│       ├── libgimpbase
│       ├── libgimpcolor
│       ├── libgimpconfig
│       ├── libgimpmath
│       ├── libgimpmodule
│       ├── libgimpthumb
│       └── libgimpwidgets
├── lib
│   ├── gimp
│   │   └── 2.0
│   │       ├── environ
│   │       ├── interpreters
│   │       ├── modules
│   │       └── plug-ins
│   │           ├── align-layers
│   │           ├── animation-optimize
│   │           ├── animation-play
│   │           ├── blinds
│   │           ├── blur
│   │           ├── border-average
│   │           ├── busy-dialog
│   │           ├── cartoon
│   │           ├── checkerboard
│   │           ├── cml-explorer
│   │           ├── color-cube-analyze
│   │           ├── color-enhance
│   │           ├── colorify
│   │           ├── colormap-remap
│   │           ├── compose
│   │           ├── contrast-retinex
│   │           ├── crop-zealous
│   │           ├── curve-bend
│   │           ├── decompose
│   │           ├── depth-merge
│   │           ├── despeckle
│   │           ├── destripe
│   │           ├── edge-dog
│   │           ├── emboss
│   │           ├── file-bmp
│   │           ├── file-cel
│   │           ├── file-compressor
│   │           ├── file-csource
│   │           ├── file-darktable
│   │           ├── file-dds
│   │           ├── file-desktop-link
│   │           ├── file-dicom
│   │           ├── file-faxg3
│   │           ├── file-fits
│   │           ├── file-fli
│   │           ├── file-gbr
│   │           ├── file-gegl
│   │           ├── file-gif-load
│   │           ├── file-gif-save
│   │           ├── file-gih
│   │           ├── file-glob
│   │           ├── file-header
│   │           ├── file-html-table
│   │           ├── file-ico
│   │           ├── file-jpeg
│   │           ├── file-pat
│   │           ├── file-pcx
│   │           ├── file-pdf-load
│   │           ├── file-pdf-save
│   │           ├── file-pix
│   │           ├── file-png
│   │           ├── file-pnm
│   │           ├── file-psd
│   │           ├── file-psp
│   │           ├── file-raw-data
│   │           ├── file-raw-placeholder
│   │           ├── file-rawtherapee
│   │           ├── file-sgi
│   │           ├── file-sunras
│   │           ├── file-svg
│   │           ├── file-tga
│   │           ├── file-tiff
│   │           ├── file-xbm
│   │           ├── file-xmc
│   │           ├── file-xwd
│   │           ├── film
│   │           ├── filter-pack
│   │           ├── flame
│   │           ├── fractal-explorer
│   │           ├── fractal-trace
│   │           ├── gfig
│   │           ├── gimpressionist
│   │           ├── goat-exercise
│   │           ├── gradient-flare
│   │           ├── gradient-map
│   │           ├── grid
│   │           ├── guillotine
│   │           ├── help
│   │           ├── hot
│   │           ├── ifs-compose
│   │           ├── imagemap
│   │           ├── jigsaw
│   │           ├── lighting
│   │           ├── mail
│   │           ├── map-object
│   │           ├── max-rgb
│   │           ├── metadata-editor
│   │           ├── metadata-viewer
│   │           ├── nl-filter
│   │           ├── pagecurl
│   │           ├── photocopy
│   │           ├── plugin-browser
│   │           ├── print
│   │           ├── procedure-browser
│   │           ├── qbist
│   │           ├── sample-colorize
│   │           ├── screenshot
│   │           ├── script-fu
│   │           ├── selection-to-path
│   │           ├── sharpen
│   │           ├── smooth-palette
│   │           ├── softglow
│   │           ├── sparkle
│   │           ├── sphere-designer
│   │           ├── tile
│   │           ├── tile-small
│   │           ├── unit-editor
│   │           ├── van-gogh-lic
│   │           ├── warp
│   │           ├── wavelet-decompose
│   │           └── web-browser
│   ├── pkgconfig
│   ├── python2.7
│   │   ├── dist-packages
│   │   └── site-packages
│   ├── python3.7
│   │   └── dist-packages
│   └── python3.8
│       └── dist-packages
├── libexec
├── man -> share/man
├── sbin
├── share
│   ├── aclocal
│   ├── applications
│   ├── ca-certificates
│   ├── fonts
│   ├── gimp
│   │   └── 2.0
│   │       ├── brushes
│   │       │   ├── Basic
│   │       │   ├── Fun
│   │       │   ├── gimp-obsolete-files
│   │       │   ├── Legacy
│   │       │   ├── Media
│   │       │   ├── Sketch
│   │       │   ├── Splatters
│   │       │   └── Texture
│   │       ├── dynamics
│   │       │   ├── Basic
│   │       │   └── FX
│   │       ├── file-raw
│   │       ├── fonts
│   │       ├── fractalexplorer
│   │       ├── gfig
│   │       ├── gflare
│   │       ├── gimpressionist
│   │       │   ├── Brushes
│   │       │   ├── Paper
│   │       │   └── Presets
│   │       ├── gradients
│   │       │   └── gimp-obsolete-files
│   │       ├── icons
│   │       │   ├── Color
│   │       │   │   ├── 24x24
│   │       │   │   │   └── apps
│   │       │   │   ├── 64x64
│   │       │   │   │   └── apps
│   │       │   │   └── scalable
│   │       │   │       └── apps
│   │       │   ├── hicolor
│   │       │   ├── Legacy
│   │       │   │   ├── 128x128
│   │       │   │   │   └── apps
│   │       │   │   ├── 12x12
│   │       │   │   │   └── apps
│   │       │   │   ├── 16x16
│   │       │   │   │   └── apps
│   │       │   │   ├── 18x18
│   │       │   │   │   └── apps
│   │       │   │   ├── 192x192
│   │       │   │   │   └── apps
│   │       │   │   ├── 20x20
│   │       │   │   │   └── apps
│   │       │   │   ├── 22x22
│   │       │   │   │   ├── apps
│   │       │   │   │   └── tools
│   │       │   │   ├── 24x24
│   │       │   │   │   └── apps
│   │       │   │   ├── 256x256
│   │       │   │   │   └── apps
│   │       │   │   ├── 32x32
│   │       │   │   │   └── apps
│   │       │   │   ├── 48x48
│   │       │   │   │   └── apps
│   │       │   │   ├── 64x64
│   │       │   │   │   └── apps
│   │       │   │   └── 96x96
│   │       │   │       └── apps
│   │       │   ├── Symbolic
│   │       │   │   ├── 24x24
│   │       │   │   │   └── apps
│   │       │   │   ├── 64x64
│   │       │   │   │   └── apps
│   │       │   │   └── scalable
│   │       │   │       └── apps
│   │       │   ├── Symbolic-High-Contrast
│   │       │   │   ├── 24x24
│   │       │   │   │   └── apps
│   │       │   │   ├── 64x64
│   │       │   │   │   └── apps
│   │       │   │   └── scalable
│   │       │   │       └── apps
│   │       │   ├── Symbolic-Inverted
│   │       │   │   ├── 24x24
│   │       │   │   │   └── apps
│   │       │   │   ├── 64x64
│   │       │   │   │   └── apps
│   │       │   │   └── scalable
│   │       │   │       └── apps
│   │       │   └── Symbolic-Inverted-High-Contrast
│   │       │       ├── 24x24
│   │       │       │   └── apps
│   │       │       ├── 64x64
│   │       │       │   └── apps
│   │       │       └── scalable
│   │       │           └── apps
│   │       ├── images
│   │       ├── menus
│   │       ├── palettes
│   │       ├── patterns
│   │       │   ├── Animal
│   │       │   ├── Fabric
│   │       │   ├── Food
│   │       │   ├── Legacy
│   │       │   ├── Paper
│   │       │   ├── Plant
│   │       │   ├── Sky
│   │       │   ├── Stone
│   │       │   ├── Water
│   │       │   └── Wood
│   │       ├── scripts
│   │       │   └── images
│   │       ├── tags
│   │       ├── themes
│   │       │   ├── Dark
│   │       │   │   └── ui
│   │       │   ├── Gray
│   │       │   │   └── ui
│   │       │   ├── Light
│   │       │   │   └── ui
│   │       │   └── System
│   │       ├── tips
│   │       ├── tool-presets
│   │       │   ├── Crop
│   │       │   ├── FX
│   │       │   ├── Paint
│   │       │   ├── Selection
│   │       │   └── Sketch
│   │       └── ui
│   │           └── plug-ins
│   ├── gtk-doc
│   │   └── html
│   │       ├── libgimp
│   │       ├── libgimpbase
│   │       ├── libgimpcolor
│   │       ├── libgimpconfig
│   │       ├── libgimpmath
│   │       ├── libgimpmodule
│   │       ├── libgimpthumb
│   │       └── libgimpwidgets
│   ├── icons
│   │   └── hicolor
│   │       ├── 16x16
│   │       │   └── apps
│   │       ├── 22x22
│   │       │   └── apps
│   │       ├── 24x24
│   │       │   └── apps
│   │       ├── 256x256
│   │       │   └── apps
│   │       ├── 32x32
│   │       │   └── apps
│   │       └── 48x48
│   │           └── apps
│   ├── locale
│   │   ├── am
│   │   │   └── LC_MESSAGES
│   │   ├── ar
│   │   │   └── LC_MESSAGES
│   │   ├── ast
│   │   │   └── LC_MESSAGES
│   │   ├── az
│   │   │   └── LC_MESSAGES
│   │   ├── be
│   │   │   └── LC_MESSAGES
│   │   ├── bg
│   │   │   └── LC_MESSAGES
│   │   ├── br
│   │   │   └── LC_MESSAGES
│   │   ├── bs
│   │   │   └── LC_MESSAGES
│   │   ├── ca
│   │   │   └── LC_MESSAGES
│   │   ├── ca@valencia
│   │   │   └── LC_MESSAGES
│   │   ├── cs
│   │   │   └── LC_MESSAGES
│   │   ├── csb
│   │   │   └── LC_MESSAGES
│   │   ├── da
│   │   │   └── LC_MESSAGES
│   │   ├── de
│   │   │   └── LC_MESSAGES
│   │   ├── dz
│   │   │   └── LC_MESSAGES
│   │   ├── el
│   │   │   └── LC_MESSAGES
│   │   ├── en_CA
│   │   │   └── LC_MESSAGES
│   │   ├── en_GB
│   │   │   └── LC_MESSAGES
│   │   ├── eo
│   │   │   └── LC_MESSAGES
│   │   ├── es
│   │   │   └── LC_MESSAGES
│   │   ├── et
│   │   │   └── LC_MESSAGES
│   │   ├── eu
│   │   │   └── LC_MESSAGES
│   │   ├── fa
│   │   │   └── LC_MESSAGES
│   │   ├── fi
│   │   │   └── LC_MESSAGES
│   │   ├── fr
│   │   │   └── LC_MESSAGES
│   │   ├── ga
│   │   │   └── LC_MESSAGES
│   │   ├── gd
│   │   │   └── LC_MESSAGES
│   │   ├── gl
│   │   │   └── LC_MESSAGES
│   │   ├── gu
│   │   │   └── LC_MESSAGES
│   │   ├── he
│   │   │   └── LC_MESSAGES
│   │   ├── hi
│   │   │   └── LC_MESSAGES
│   │   ├── hr
│   │   │   └── LC_MESSAGES
│   │   ├── hu
│   │   │   └── LC_MESSAGES
│   │   ├── id
│   │   │   └── LC_MESSAGES
│   │   ├── is
│   │   │   └── LC_MESSAGES
│   │   ├── it
│   │   │   └── LC_MESSAGES
│   │   ├── ja
│   │   │   └── LC_MESSAGES
│   │   ├── ka
│   │   │   └── LC_MESSAGES
│   │   ├── kk
│   │   │   └── LC_MESSAGES
│   │   ├── km
│   │   │   └── LC_MESSAGES
│   │   ├── kn
│   │   │   └── LC_MESSAGES
│   │   ├── ko
│   │   │   └── LC_MESSAGES
│   │   ├── ky
│   │   │   └── LC_MESSAGES
│   │   ├── lt
│   │   │   └── LC_MESSAGES
│   │   ├── lv
│   │   │   └── LC_MESSAGES
│   │   ├── mk
│   │   │   └── LC_MESSAGES
│   │   ├── ml
│   │   │   └── LC_MESSAGES
│   │   ├── mr
│   │   │   └── LC_MESSAGES
│   │   ├── ms
│   │   │   └── LC_MESSAGES
│   │   ├── my
│   │   │   └── LC_MESSAGES
│   │   ├── nb
│   │   │   └── LC_MESSAGES
│   │   ├── nds
│   │   │   └── LC_MESSAGES
│   │   ├── ne
│   │   │   └── LC_MESSAGES
│   │   ├── nl
│   │   │   └── LC_MESSAGES
│   │   ├── nn
│   │   │   └── LC_MESSAGES
│   │   ├── oc
│   │   │   └── LC_MESSAGES
│   │   ├── pa
│   │   │   └── LC_MESSAGES
│   │   ├── pl
│   │   │   └── LC_MESSAGES
│   │   ├── pt
│   │   │   └── LC_MESSAGES
│   │   ├── pt_BR
│   │   │   └── LC_MESSAGES
│   │   ├── ro
│   │   │   └── LC_MESSAGES
│   │   ├── ru
│   │   │   └── LC_MESSAGES
│   │   ├── rw
│   │   │   └── LC_MESSAGES
│   │   ├── si
│   │   │   └── LC_MESSAGES
│   │   ├── sk
│   │   │   └── LC_MESSAGES
│   │   ├── sl
│   │   │   └── LC_MESSAGES
│   │   ├── sr
│   │   │   └── LC_MESSAGES
│   │   ├── sr@latin
│   │   │   └── LC_MESSAGES
│   │   ├── sv
│   │   │   └── LC_MESSAGES
│   │   ├── ta
│   │   │   └── LC_MESSAGES
│   │   ├── te
│   │   │   └── LC_MESSAGES
│   │   ├── th
│   │   │   └── LC_MESSAGES
│   │   ├── tr
│   │   │   └── LC_MESSAGES
│   │   ├── tt
│   │   │   └── LC_MESSAGES
│   │   ├── uk
│   │   │   └── LC_MESSAGES
│   │   ├── vi
│   │   │   └── LC_MESSAGES
│   │   ├── xh
│   │   │   └── LC_MESSAGES
│   │   ├── yi
│   │   │   └── LC_MESSAGES
│   │   ├── zh_CN
│   │   │   └── LC_MESSAGES
│   │   ├── zh_HK
│   │   │   └── LC_MESSAGES
│   │   └── zh_TW
│   │       └── LC_MESSAGES
│   ├── man
│   │   ├── man1
│   │   └── man5
│   └── metainfo
└── src

465 directories
```

He usado la orden...
```
tree -d /usr/local/
```
...sólo mostrando directorios, porque ya es suficientemente largo el output como para mostrar ficheros también. 

Pero para que nos hagamos una idea, aquí tenemos nuestros binarios ejecutables generados, por ejemplo:
```
vagrant@mimaquina:/usr/local/bin$ ls -la 
total 52496
drwxr-xr-x  2 root root     4096 Nov  3 06:00 .
drwxr-xr-x 11 root root     4096 Nov  3 05:49 ..
lrwxrwxrwx  1 root root        9 Nov  3 06:00 gimp -> gimp-2.10
-rwxr-xr-x  1 root root 53624584 Nov  3 06:00 gimp-2.10
-rwxr-xr-x  1 root root    50760 Nov  3 06:00 gimp-test-clipboard-2.0
-rwxr-xr-x  1 root root    67328 Nov  3 06:00 gimptool-2.0
```

Hasta aquí, podemos considerar que tenemos gimp instalado.

## Tercera parte

### **Desinstalación**

No viene explícitamente dicho en ningún documento dentro del código fuente, o al menos yo no lo encontré donde lo dice, pero para la desinstalación tenemos disponible la opción `uninstall`.

Esto lo sé por 2 razones:
* Puedo ejecutar un `sudo make uninstall` y ver que efectivamente, ha funcionado 
* En el fichero Makefile generado personalizado para nuestra instalación, tenemos declarada una fase `uninstall`...
     ```
     am__uninstall_files_from_dir = { \
     test -z "$$files" \
     || { test ! -d "$$dir" && test ! -f "$$dir" && test ! -r "$$dir"; } \
     || { echo " ( cd '$$dir' && rm -f" $$files ")"; \
          $(am__cd) "$$dir" && rm -f $$files; }; \
     }
     ```
     Entiendo que es ese el bloque de código que lo define, ya que está borrando, pero hay otras partes en el código completo que también mencionan el `uninstall`

Por todo esto, puedo estar bastante seguro de que eso va a funcionar.

Efectivamente, si lo ejecuto funciona, y borra lo correspondiente del raíz `/usr/local`.

Por ejemplo, nos damos cuenta de que ya no tenemos los binarios...
```
vagrant@mimaquina:~/gimp-2.10.22$ ls -la /usr/local/bin/
total 8
drwxr-xr-x  2 root root 4096 Nov  3 06:15 .
drwxr-xr-x 11 root root 4096 Nov  3 05:49 ..
```

Eso sí, es importante anotar que la estructura de directorios que previamente generó en `/usr/local` la mantiene, **sólo borra ficheros**.  

¿Por qué hace esto? Sinceramente no lo sé, supongo que para futuras instalaciones, no tener que volver a generar toda la estructura de directorios y ahorrar así algo de tiempo.