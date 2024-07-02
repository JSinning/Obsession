Deplegamos maquina de dockerlabs con

`sudo ./auto_deploy.sh obsession.tar`

hacemos un brarido con nmap para lista los puertos abiertios usando 

`sudo nmap -p- --open -sS -sCV --min-rate 5000 -n -Pn -vvv 172.17.0.2 -oN scaning.txt`

![Pasted image 20240701113645](https://github.com/JSinning/Obsession/assets/43380469/eb1edaca-0461-480c-8e7e-62ed941d6480)

![Pasted image 20240701113722](https://github.com/JSinning/Obsession/assets/43380469/ee8c5580-a877-435d-8d7a-c5b73f92a144)


Ok lugo de hacer el barido de puertos onsevamos que la maquina tiene abiertos el puerto **21, 22, 80** puertos comunes que ya conecemos. Analisando cada puerto obesrvamos que puerto 21 **ftp** tiene la el usaurio Anonymous el cual nos permite listar y ver la alguanos achivos que encuentran compratido. por otro lado tenemos el puerto 21 **SSH** con una version de **OpenSSH  9.6p1** muy actual  la cual nos permite usar exploits para enuemar usaurios , y finalmente  el puerto 80 **http** en el cual vemos en que tien un titulo llamdo  **Russoki coaching**  en su sitio web ya que es un **APACHE  2.4.58**

comenzemos analizando el puerto ftp usando el comando 
 **`ftp 172.17.0.2`** 
 y luego porpocioanado el usarrio Anonymous y dejamis la contraseña en blanco.
 ![Pasted image 20240701114919](https://github.com/JSinning/Obsession/assets/43380469/ecbc9de7-6262-4931-bda2-f07d1cc97f5f)

 usando el comando `ls` vemos la lista de los archivos que se encuentra en el servido y con el **`mget`** comado reservado para ftp descargamos todo estos archovos a nuesta maquina para anlizarlos.
 ![Pasted image 20240701115202](https://github.com/JSinning/Obsession/assets/43380469/7d8d3386-9c10-4e30-80a5-5b82c5e747fd)

 El priemer archivo que analizamos es chat-gonza.txt y vemos una charla muy estraña
![Pasted image 20240701121334](https://github.com/JSinning/Obsession/assets/43380469/4d2ffaa9-89c4-4a2a-a220-7b3ff6802bf5)

 pero podemos ver dos posibles usurios **Gonza**  y **Russoski** los tendremos en cuenta  para mas adelante.
 luego revisamos el otro archivo  y notamos que es un  lista por hacer. 
![Pasted image 20240701184748](https://github.com/JSinning/Obsession/assets/43380469/e1718c9c-a02e-406c-8637-26cd72200ab1)


es interasante el ultimo punto donde vemos que dice que en su equipo tiene ciertos permos avilitados pero de igualmanera lo tendremos en cuenta para mas adelante.

ahora teniendo lo que aremos es revisar el sito web 
![Pasted image 20240702092519](https://github.com/JSinning/Obsession/assets/43380469/e18e8c4e-e133-4da4-b38e-384cad47f324)

 vemos que una paguina estatica donde nos habla acerca de de su profesion. Al final del sitio podemos ver un formulario para asesorias entrnamientos personales iintenaremos llenar los campo y veamos a donde nos llevan.
![Pasted image 20240702093017](https://github.com/JSinning/Obsession/assets/43380469/7cd28b99-6f7c-43ca-ad0f-46cdd17c4499)

al regustranos no conseguimos mucho solo conaseguimos un paguin que nos dice que pronto nos conatactaran asi que procedemos a use wfuzz para fuzzing al sito web  y encntra nuevas rutas.

`wfuzz -c -t 200 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt --hc 404 http://172.17.0.2/FUZZ`
![Pasted image 20240702094815](https://github.com/JSinning/Obsession/assets/43380469/af8a4d40-71e0-42d6-af07-d98abcdc9c86)


vemos que encontramos alguan rutras  como lo es **backup** e **important** investigaremos cada una de ellas.

![Pasted image 20240702095359](https://github.com/JSinning/Obsession/assets/43380469/c43bbd6f-e25e-49cd-aa23-d5c9d21ec840)


![Pasted image 20240702095436](https://github.com/JSinning/Obsession/assets/43380469/4a094af8-cb75-423a-ad9f-93b255cfb552)

 lo encontra en backup es intersante nos confirma que el usuariop **Russoski** exite en el sistema y lo otro es el manifiesto d hacker un fracmento del libro muy popular.

ahora si nos concentraremos en el usauario **Russoski** y usando la heramienta **hydra** haremos un brutforoce para consegir la contaseña del usuario.

 `hydra -l russoski -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2` 
 
en este caso usaromos el el diccionario rockyou para ver si alguan de la contraseñas son validas.

![Pasted image 20240702100103](https://github.com/JSinning/Obsession/assets/43380469/abb95df2-208b-4c48-9b10-8ef38cfdd907)


finalmenet lo tenemos el usario **russoski** tine como password **iloveme**  ahora usando SSH iniciamos secion en la maquina.

`ssh russoski@172.17.0.2` 

luego proporcionamos la password y estamod dentro.

![Pasted image 20240702100413](https://github.com/JSinning/Obsession/assets/43380469/8c47990e-2a19-49aa-b013-33be9495f090)


estamos con el usurio russoski ahora necesitamo escalar nuestro privilegio. para eso usaremos el comando `sudo -l` para averioguar si el usario ejecuta algo como root

![Pasted image 20240702100634](https://github.com/JSinning/Obsession/assets/43380469/1059da3e-5468-45c7-b3b2-5b71f339f32f)


y si el usuario puede ejecutar vim sin porporcionar contraseña  como el usurio root. ahora nos dijimos a la paguina GTFOBins pagina que nos puede ser utilidad para averiguar como excar priviligios con los comando que se encuentran en sudores.

![Pasted image 20240702101123](https://github.com/JSinning/Obsession/assets/43380469/3d9d94db-1b34-4d85-b6d7-81c3b1e9ab1b)


si vemos para el conado sudo tenemos esta propiedades para escalar nuestro privileguion en nuestro caso usarmos la primera que nos expona un shell.

`sudo -u root /usr/bin/vim -c ':!/bin/bash'`

este el condo que nosotri usaremos.

y finalmente lo conseguimos la y somo root.

![Pasted image 20240702101457](https://github.com/JSinning/Obsession/assets/43380469/c2323723-2796-4344-ae5b-911de25757e6)



`by juandas13` 





 

 
