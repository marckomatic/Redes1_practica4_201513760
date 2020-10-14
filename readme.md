<ul>
<li> Marco Pablo Mazariegos Macario </li>
<li> 201513760 </li>
<li> Redes de computadoras 1 </li>
</ul>

<h1> Manual de Práctica 4 </h1>

<p> Se necesita crear una red con el concepto de VLANs para gestionar el trafico
de datos en las areas de ventas y contabilidad de la empresa.</p>
<p> La topología propuesta es la siguiente: </p>
<img src='/images/img1.png'>
<p>Habra una capa de distribucion compuesta por 3 EtherSwitch y un router. 
Además de una capa de accesa compuesta por switches capa 2.  <p>

<h2> Requerimientos </h2>

<ul>
    <li>Software GNS3 en cualquiera de sus versiones</li>
    <li> Archivo de imagen de Router c3725 </li>
    <li> Sowtware de captura de paquetes disponible en  
        <a href='https://wireshark.com/index.php?/'> 
            https://wireshark.com</a>  </li>
    <li>Vmware Player o Workstation</li>
    <li>Maquina virtual con cualquier distribucion de Linux </li>
</ul>

<h2> Configuración de Topología</h2>

<p> Con la topología ya diagramada en GNS3, debemos configurar los EtherSwitches para 
que todas las interfaces de cada uno esten en modo Trunk.</p>

<p> Para ello, vamos a ingresar los siguientes comandos en cada uno de los EtherSwitches:</p>


<ol>
    <li>configure terminal</li>
    <li>interface range fastEthernet 1/0 - 9</li>
    <li>switchport trunk mode </li>
    <li>exit </li>
    <li>exit </li>
</ol>
<img src='/images/img2.png'>
<img src='/images/img3.png'>
<img src='/images/img4.png'>

<p> Ahora, configuraremos los port-channel en las interfaces con conexion paralela.
Esta configuracion debe hacerce en los dos extremos de cada port channel. 
Esto en los ESW1, ESW2 y ESW3</p>

<p>Para configurar el port-channel, escribiremos los comandos: </p>
<ol>
    <li>configure terminal</li>
    <li>interface fastEthernet n/m</li>
    <li>interface channel-group y mode on</li>
    <li>exit </li>
    <li>exit </li>
</ol>

<p>Podemos ver los channel creados con el comando:</p>
<ul>
    <li>show etherchannel port-channel</li>
</ul>

<img src='/images/img5.png'>

<p> Ahora, resta configurar el ESW1 como servidor con los siguientes comandos comandos: </p>
<ol>
    <li>vlan database</li>
    <li>vtp domain redes1_201513760</li>
    <li>vtp password redes_1</li>
    <li>vtp v2-mode</li>
    <li>vtp server</li>
    <li>exit </li>
</ol>

<img src='/images/img6.png'>

<p>Los ESW2 y ESW3 se deben configurar como clientes: </p>

<p> Ahora, resta configurar el ESW1 como servidor con los siguientes comandos comandos: </p>
<ol>
    <li>vlan database</li>
    <li>vtp domain redes1_201513760</li>
    <li>vtp password redes_1</li>
    <li>vtp v2-mode</li>
    <li>vtp server</li>
    <li>exit </li>
</ol>

<img src='/images/img7.png'>
<img src='/images/img8.png'>

<p> Para configurar las VLANs en el ESW1 (el serividor), ingresamos los siguientes comandos.
<ol>
    <li>vlan database</li>
    <li>vlan 10 name VENTAS</li>
    <li>vlan 20 name CONTABILIDAD</li>
</ol>

<img src='/images/img9.png'>

<p>Podemos verificar las VLANs creadas en el servidor con el comando. </p>
<ul>
    <li>show vlan-switch</li>
</ul>

<img src='/images/img10.png'>


<p> 
<table>
<tr>
<td>VLAN Ventas</td> <td>VLAN CONTABILIDAD</td> 
</tr>
<tr>
<td>Gateway: 192.168.10.254 </td><td>Gateway: 192.168.20.254 </td>
</tr>
<tr>
<td>Mascara: 255.255.255.0</td><td>Mascara: 255.255.255.0</td> 
</tr>
<tr>
<td>Sub-Interface: 0/0.10 </td><td>Sub-Interface: 0/0.20</td> 
</tr>
</table>

<P> Para configurarlo, en el router R1 debemos configurar las subinterfaces.
Esto lo hacemos con los comandos: </P>

<ol>
    <li> configure terminal </li>
    <li> interface fastEthernet n/m.x </li>
    <li> encapsulation dot1Q X</li>
    <li> ip address [gateway] [mask] </li>
    <li> exit </li>
    <li> exit </li>
</ol>

<img src='/images/img11.png'>
<img src='/images/img12.png'>

<p> Ahora, debemos configurar los switch capa 2 para que los puertos sean modo access
y tenga la vlan a la que esta asignada el terminal conectado al puerto.</p>

<p> Los switch1 y switch2 quedarian configurados asi: </p>
<img src='/images/img14.png'>

<p> Probamos hacer ping desde la maquina linux y una vpc para ver si podemos hacer 
contacto con todos los equipos en la red </p>
<img src='/images/img15.png'>
<img src='/images/img16.png'>

<h2>Spanning tree </h2>

<p>El spanning tree es un protocolo de distribucion de datos para la funcion
optima de las VLANs a nivel de la capa 3. Los Etherswitch configuran por 
default un modo spanning tree que permiten que el enviado de paquetes no se encicle
y pueda tambien recuperarse la conexion si no se fallan.</p>

<h3> Root bridge </p>
<p>Se puede determinar en una VLAN cual es el dispositivo Root del spanning tree
ejecutando el siguiente comando en cada uno de los etherswitches: </p>

<ul>
    <li>show spanning-tree root </li>
</ul>

<p>El comando muestra el estado del puente o bridge que tiene en cada VLAN,
si el dispositivo es root en alguna de ellas, se mostrara en la salida de la ejecucion del comando el resultado <b>this bridge is root.</b> <p>

<p>En el caso de nuestra red, el <n> ESW1 es el root </p>
<img src='/images/img17.png'>

<h4>Blocked ports </p>

<p>El spanning-tree protocol, cuando existe posibilidad de paquetes enciclados 
en la red, blockea puertos/interfaces en cada dispositivo para evitar que esto ocurre.</p>

<p>Para obtener los puertos bloqueados en cada etherswitch, dembos ingresar el comando: </p>

<ul>
    <li>show spanning-tree blockedports</li>
</ul>

<p> Puertos bloqueados en ESW1 </p>
<img src='/images/img18.png'>

<p> Puertos bloqueados en ESW2 </p>
<img src='/images/img19.png'>

<p> Puertos bloqueados en ESW3 </p>
<img src='/images/img20.png'>

<p> Las interfaces bloqueadas (para todas las VLANs) en los ESW son <p>

<ul>
    <li>fa 1/7 ESW2</li>
    <li>fa 1/7 ESW3 </li>
    <li>PortChannel 3 ESW3 </li>
</ul>

<h2>Diagrama de rutas principales </h2>
<p> 
El diagrama de rutas principales es el diagrama que muestra las interfaces no bloqueadas
por el spanning-tree. El diagrama de rutas principales para nuestra red es el siguiente.
</p>

<img src='/images/img20.png'>
<h2>Captura de paquetes</h2>
<p>Para comprobar que el diagrama de rutas principales, podemos
poner un packet watcher en una interface bloqueada y ver si pasan paquetes
por esa interface. En este caso, la pondremos en el portchannel 3 
(interfaces fa1/5 y fa1/6 del ESW2 y ESW3). </p>

<img src='/images/img22.png'>

<p>Haciendo ping desde la maquina 192.168.20.10, hacia 192.168.20.20 vemos que no reconoce ningún paquete por esa ruta. </p>

<img src='/images/img23.png'>

<img src='/images/img24.png'>

<p>Sin embargo, si movemos el punto de captura en el portchannel 1 (interfaces fa 1/1 y fa 1/2 entre el ESW1 y ESW2), y hacemos ping desde 192.168.20.10 hacia 192.168.10.20. </p>

<img src='/images/img25.png'>

<p> Encontramos que la interfaz port-channel 1 si es transitada por paquetes. </p>

<img src='/images/img26.png'>
