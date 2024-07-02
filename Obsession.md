Deplegamos maquina de dockerlabs con

`sudo ./auto_deploy.sh obsession.tar`

hacemos un brarido con nmap para lista los puertos abiertios usando 

`sudo nmap -p- --open -sS -sCV --min-rate 5000 -n -Pn -vvv 172.17.0.2 -oN scaning.txt`

![[Pasted image 20240701113645.png]]

![[Pasted image 20240701113722.png]]

Ok lugo de hacer el barido de puertos onsevamos que la maquina tiene abiertos el puerto **21, 22, 80** puertos comunes que ya conecemos. Analisando cada puerto obesrvamos que puerto 21 **ftp** tiene la el usaurio Anonymous el cual nos permite listar y ver la alguanos achivos que encuentran compratido. por otro lado tenemos el puerto 21 **SSH** con una version de **OpenSSH  9.6p1** muy actual  la cual nos permite usar exploits para enuemar usaurios , y finalmente  el puerto 80 **http** en el cual vemos en que tien un titulo llamdo  **Russoki coaching**  en su sitio web ya que es un **APACHE  2.4.58**

comenzemos analizando el puerto ftp usando el comando 
 **`ftp 172.17.0.2`** 
 y luego porpocioanado el usarrio Anonymous y dejamis la contraseña en blanco.
 ![[Pasted image 20240701114919.png]]
 usando el comando `ls` vemos la lista de los archivos que se encuentra en el servido y con el **`mget`** comado reservado para ftp descargamos todo estos archovos a nuesta maquina para anlizarlos.
 ![[Pasted image 20240701115202.png]]
 El priemer archivo que analizamos es chat-gonza.txt y vemos una charla muy estraña
 ![[Pasted image 20240701121334.png]]
 pero podemos ver dos posibles usurios **Gonza**  y **Russoski** los tendremos en cuenta  para mas adelante.
 luego revisamos el otro archivo  y notamos que es un  lista por hacer. 
 ![[Pasted image 20240701184748.png]]

es interasante el ultimo punto donde vemos que dice que en su equipo tiene ciertos permos avilitados pero de igualmanera lo tendremos en cuenta para mas adelante.

ahora teniendo lo que aremos es revisar el sito web 
![[Pasted image 20240702092519.png]]
 vemos que una paguina estatica donde nos habla acerca de de su profesion. Al final del sitio podemos ver un formulario para asesorias entrnamientos personales iintenaremos llenar los campo y veamos a donde nos llevan.
 ![[Pasted image 20240702093017.png]]
al regustranos no conseguimos mucho solo conaseguimos un paguin que nos dice que pronto nos conatactaran asi que procedemos a use wfuzz para fuzzing al sito web  y encntra nuevas rutas.

`wfuzz -c -t 200 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt --hc 404 http://172.17.0.2/FUZZ`
![[Pasted image 20240702094815.png]]

vemos que encontramos alguan rutras  como lo es **backup** e **important** investigaremos cada una de ellas.

![[Pasted image 20240702095359.png]]

![[Pasted image 20240702095436.png]]
 lo encontra en backup es intersante nos confirma que el usuariop **Russoski** exite en el sistema y lo otro es el manifiesto d hacker un fracmento del libro muy popular.

ahora si nos concentraremos en el usauario **Russoski** y usando la heramienta **hydra** haremos un brutforoce para consegir la contaseña del usuario.

 `hydra -l russoski -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2` 
 
en este caso usaromos el el diccionario rockyou para ver si alguan de la contraseñas son validas.

![[Pasted image 20240702100103.png]]

finalmenet lo tenemos el usario **russoski** tine como password **iloveme**  ahora usando SSH iniciamos secion en la maquina.

`ssh russoski@172.17.0.2` 

luego proporcionamos la password y estamod dentro.

![[Pasted image 20240702100413.png]]

estamos con el usurio russoski ahora necesitamo escalar nuestro privilegio. para eso usaremos el comando `sudo -l` para averioguar si el usario ejecuta algo como root

![[Pasted image 20240702100634.png]]

y si el usuario puede ejecutar vim sin porporcionar contraseña  como el usurio root. ahora nos dijimos a la paguina GTFOBins pagina que nos puede ser utilidad para averiguar como excar priviligios con los comando que se encuentran en sudores.

![[Pasted image 20240702101112.png]]

si vemos para el conado sudo tenemos esta propiedades para escalar nuestro privileguion en nuestro caso usarmos la primera que nos expona un shell.

`sudo -u root /usr/bin/vim -c ':!/bin/bash'`

este el condo que nosotri usaremos.

y finalmente lo conseguimos la y somo root.

![[Pasted image 20240702101457.png]]


`by juandas13` 





 

 