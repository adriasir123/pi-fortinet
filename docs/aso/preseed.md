# Preseed


lista de locales: /usr/share/i18n/SUPPORTED

sudo apt install libarchive-tools (para instalar bsdtar)

bsdtar -C prueba -xf debian-11.5.0-amd64-netinst.iso



genisoimage -r -J -b isolinux/isolinux.bin -c isolinux/boot.cat \                                                    
            -no-emul-boot -boot-load-size 4 -boot-info-table \                                                           
            -o preseed-debian-10.2.0-i386-netinst.iso isofiles



chmod +w -R isolinux/ (necesario para generar la ISO porque necesita permisos ahí)



## 





























