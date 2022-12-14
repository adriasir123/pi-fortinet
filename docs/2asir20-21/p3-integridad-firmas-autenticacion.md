---
title: "Práctica 3: Integridad, firmas y autenticación"
---

# Tarea 1: Firmas electrónicas

## 1. Manda un documento y la firma electrónica del mismo a un compañero. Verifica la firma que tú has recibido.

Este es mi documento
```
-rw-r--r-- 1 vagrant vagrant   14 Oct 28 12:09 file.txt
```

Genero la firma para él
```
gpg --output file.txt.sig --detach-sig file.txt
```

Y ésta es la firma resultante
```
-rw-r--r-- 1 vagrant vagrant  438 Oct 28 12:17 file.txt.sig
```

Fichero y firma, se los mando a Calderón.

------------------------


El fichero y firma que Calderón me manda a mí, son los siguientes

``` 
-rw-r--r-- 1 vagrant vagrant  438 Oct 28 12:41 DocumentoImportanteParaAdrian.sig
-rw-r--r-- 1 vagrant vagrant   31 Oct 28 12:41 DocumentoImportanteParaAdrian.txt
```

Verifico esa firma específica de ese fichero con

```
gpg --verify DocumentoImportanteParaAdrian.sig DocumentoImportanteParaAdrian.txt
```

> Importante el orden de nombres de ficheros en el comando anterior. Primero debemos escribir la `detached signature` y segundo, el fichero original del que se generó esa firma.   

```
gpg: Signature made Tue 27 Oct 2020 11:12:50 AM GMT
gpg:                using RSA key DCFB091C5495684E59BC061EA52A681834F0E596
gpg: Good signature from "José Miguel Calderón Frutos <josemiguelcalderonfrutos@gamil.com>" [unknown]
gpg: WARNING: This key is not certified with a trusted signature!
gpg:          There is no indication that the signature belongs to the owner.
Primary key fingerprint: DCFB 091C 5495 684E 59BC  061E A52A 6818 34F0 E596
```

Para que este proceso haya funcionado, tengo que haber tenido primero la clave pública de Calderón asociada a esa firma en mi anillo de claves.

Así, se podrá completar este proceso de desencriptación del hash del fichero original (a eso es a lo que llamamos firma, al hash de un fichero cifrado con una clave privada).


## 2. ¿Qué significa el mensaje que aparece en el momento de verificar la firma?

```
gpg: Firma correcta de "Pepe D <josedom24@gmail.com>" [desconocido]
gpg: ATENCIÓN: ¡Esta clave no está certificada por una firma de confianza!
gpg:          No hay indicios de que la firma pertenezca al propietario.
Huellas dactilares de la clave primaria: E8DD 5DA9 3B88 F08A DA1D  26BF 5141 3DDB 0C99 55FC
```



































# Tarea 2: Correo seguro con evolution/thunderbird



# Tarea 3: Integridad de ficheros



# Tarea 4: Integridad y autenticidad (apt secure)




# Tarea 5: Autentificación: ejemplo SSH
