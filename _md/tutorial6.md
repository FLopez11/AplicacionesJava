# Uso de JSTL

La tecnología JavaServer Pages Standard Tag Library (JSTL) es un componente de Java EE. Extiende las ya conocidas JavaServer Pages (JSP) proporcionando cuatro bibliotecas de etiquetas (Tag Libraries) con utilidades ampliamente utilizadas en el desarrollo de páginas web dinámicas.

Estas bibliotecas de etiquetas extienden de la especificación de JSP (la cual a su vez extiende de la especificación de Servlet). Su API permite además desarrollar bibliotecas propias de etiquetas.

Las bibliotecas englobadas en JSTL son:

* core: iteraciones, condicionales, manipulación de URL y otras funciones generales.
* xml: para la manipulación de XML y para XML-Transformation.
* sql: para gestionar conexiones a bases de datos.
* fmt: para la internacionalización y formateo de las cadenas de caracteres como cifras.

## Creación de la base de datos

Para este ejemplo se utilizará nuevamente la base de datos de prueba, llamada *universidad.db*, utilizando *SQLite*. Es posible utilizar diferentes herramientas para crear bases de datos en este formato, entre ellas se encuentra el *plugin* para Firefox llamado *SQLite Manager*, o bien, la herramienta *SQLiteBrowser*.

Por el momento solo se utilizará una tabla con información de profesores de una universidad. El código SQL para crear dicha tabla sería el siguiente:

	CREATE TABLE "profesor" (
	  id INTEGER PRIMARY KEY ASC,  
	  cedula VARCHAR,  
	  nombre VARCHAR,  
	  titulo VARCHAR,  
	  area VARCHAR,  
	  telefono VARCHAR
	);
 
Luego de crear la base de datos se agregaron algunos datos de ejemplo a esta tabla. Las siguientes instrucciones SQL permiten poblar la tabla alguna información:

	INSERT INTO profesor (id,cedula,nombre,titulo,area,telefono) 
	  VALUES (1,'101110111','Carlos Perez Rojas','Licenciado',
	            'Administracion','3456-7890');
	INSERT INTO profesor (id,cedula,nombre,titulo,area,telefono) 
	  VALUES (2,'202220222','Luis Torres','Master',
	            'Economia','6677-3456');
	INSERT INTO profesor (id,cedula,nombre,titulo,area,telefono) 
	  VALUES (3,'303330333','Juan Castro','Licenciado',
	            'Matematica','67455-7788');

## Página de listado de profesores

La primer página consistirá del listado de todos los profesores que se encuentran en la base de datos. El código que se muestra a continuación (llamado *listaProfesores.jsp*) establece la conexión con la base de datos (utilizado *JDBC*), ejecuta la instrucción de consulta, recupera los datos y crea una tabla HTML con la información obtenida.

	<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
	<%@ taglib uri="http://java.sun.com/jsp/jstl/sql" prefix="sql" %>
	
	<sql:setDataSource dataSource="universidad"></sql:setDataSource>
	<sql:query var="profesores">
	    select * from profesor
	</sql:query>
	<html>
	  <head>
	    <title>Sistema Universitario</title>
	    <link rel="stylesheet" href="style.css">
	  </head>
	  <h1>Sistema Universitario</h1>
	  <h2>Listado de profesores</h2>
	  <table>
	    <thead>
	    <tr><th>Cedula</th><th>Nombre</th>
	    <th>Titulo</th><th>Acciones</th></tr>
	    </thead>
	    <tbody>
	    <c:forEach var="profesor" begin="0" items="${profesores.rows}">
	       <tr><td>${profesor.cedula}</td>
	        <td>${profesor.nombre}</td>
	        <td>${profesor.titulo}</td>
	        <td><a href='/detalleProfesor.jsp?id=${profesor.id}'>
	              <input type="submit" value="Detalle"/></a>
	            <a href='/eliminarProfesor.jsp?id=${profesor.id}'>
	              <input type="submit" value="Eliminar"/></a></td></tr>
	    </c:forEach>
	   </tbody>
	   <tfoot>
	    <tr><td><a href='/agregarProfesor.jsp'>
	        <input type="submit" name="action" value="Agregar"/></a>
	    </td><td></td><td></td><td></td></tr>
	   </tfoot>
	  </table>
	</html>

Es importante observar en este código que existen enlaces que accederán a otras páginas para realizar acciones con los datos. Por ejemplo, el enlace de "Detalle" ejecutará la página *detalleProfesor.jsp* con el parámetro *ID*; y el enlace de "Eliminar" ejecutará la página *eliminarProfesor.jsp* con el mismo parámetro *ID*. También está presente otro enlace "Agregar" que invocará a la página *agregarProfesor.jsp* pero sin parámetros.

## Detalle del profesor

La página *detalleProfesor.jsp* recibe como parámetro el *ID* de un profesor, recupera sus datos desde la base y datos, y los muestra en un formulario HTML. La estructura general de la consulta a la base de datos es muy similar a la anterior con la diferencia que se recupera solo un registro particular.

	<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
	<%@ taglib uri="http://java.sun.com/jsp/jstl/sql" prefix="sql" %>
	
	<sql:setDataSource dataSource="universidad"></sql:setDataSource>
	<sql:query var="profesores">
	    select * from profesor where id = ?
	    <sql:param value="${param.id}" />
	</sql:query>
	<html>
	  <head>
	    <title>Sistema Universitario</title>
	    <link rel="stylesheet" href="style.css">
	  </head>
	  <h1>Sistema Universitario</h1>
	  <h2>Detalle de Profesor</h2>
	  <form name="ActualizarProfesor" action="actualizarProfesor" method="get">
	  <table style="width:400px;">
	    <thead>
	    <tr><th></th><th></th></tr>
	    </thead>
	    <tbody>
	     <c:forEach var="profesor" begin="0" items="${profesores.rows}">
	    <input type="hidden" name="id" value="${profesor.id}"/>
	    <tr><td>Nombre:</td><td>
	      <input type="text" name="nombre" value="${profesor.nombre}"/></td></tr>
	    <tr><td>Cedula:</td><td>
	      <input type="text" name="cedula" value="${profesor.cedula}"/></td></tr>
	    <tr><td>Titulo:</td><td>
	      <input type="text" name="titulo" value="${profesor.titulo}"/></td></tr>
	    <tr><td>Area:</td><td>
	      <input type="text" name="area" value="${profesor.area}"/></td></tr>
	    <tr><td>Telefono:</td><td>
	      <input type="text" name="telefono" value="${profesor.telefono}"/></td></tr>
	    </tbody>
	    <tfoot>
	    <tr><td><input type="submit" value="Actualizar" /></td><td></td></tr>
	    </c:forEach>
	    </tfoot>
	   </tbody>
	  </table>
	  </form>
	</html>
	
Este código también cuenta con un enlace adicional "Actualizar" que permite tomar la información del formulario y realizar la actualización de datos en la base de datos, mediante la página *actualizarProfesor.jsp*

## Hoja de estilo

Con el fin de mejorar la apariencia de este ejemplo se ha desarrollado una pequeña hoja de estilo (css) que permite organizar mejor los datos de la tabla y el formulario. El archivo llamado *style.css* cuenta con el siguiente código:

	body {
	  line-height: 1.6em;
	  font-family:"Lucida Sans Unicode", "Lucida Grande", Sans-Serif;
	  color: #009;
	}
	table {
	  font-size: 12px;
	  margin: 45px;
	  width: 480px;
	  text-align: left;
	  border-collapse: collapse;
	}
	th, tfoot td {
	  font-size: 13px;
	  font-weight: normal;
	  padding: 8px;
	  background: #b9c9fe;
	  border-top: 4px solid #aabcfe;
	  border-bottom: 1px solid #fff;
	  color: #039;
	  text-align: center;
	}
	td {
	  padding: 8px;
	  background: #e8edff;
	  border-bottom: 1px solid #fff;
	  color: #669;
	  border-top: 1px solid transparent;
	}
	tr:hover td {
	  background: #d0dafd;
	  color: #339;
	}
	input[type="text"] {
	  width: 250px;
	}
	
## Ambiente de ejecución

Para ejecutar este ejemplo es necesario contar con un *servidor de servlets* que permita también la ejecución de plantillas *JSTL*. La herramienta más utilizada para esto es el *Apache Tomcat*, el cuál es muy potente y cuenta con gran cantidad de parámetros de configuración. Sin embargo, para propósito de desarrollo y depuración de programas basta con un ambiente más liviano tal como *Winstone*.

*Winstone* consiste de un único archivo de menos de 350 KB, llamado *winstone-0.9.10.jar*, el cual puede ser ejecutado directamente mediante Java. Sin embargo, poder utilizar plantillas JSP se requiere de la herramienta *Jasper* que consiste de múltiples librerías adicionales.

Para acceder a la base de datos *SQLite*, mediante *JDBC*, es necesario contar con una librería que incluya el driver adecuado. Aún cuando existen diferentes librerías que hacen esto, ninguna es pequeña.

### Estructura de directorios

La ubicación de los diferentes archivos de código, y librerías se muestra en el siguiente esquema de directorios:

	tutorial6
	  run.bat
	  winstone-0.9.10.jar
	  database
	    universidad.db
	  root
	    style.css
	    listaProfesores.jsp
	    detalleProfesor.jsp
	  lib
	    el-api-6.0.18.jar
	    jasper-6.0.18.jar
	    jasper-el-6.0.18.jar
	    jsp-api-6.0.18.jar
	    jstl-impl-1.2jar
	    jtds-1.2.4.jar
	    juli-6.0.18.jar
	    servlet-api-2.5.jar
	    sqlite-jdbc-3.5.9.jar
    
## Ejecución del ejemplo
 
Teniendo instalado el JDK de Java (no el JRE) basta con ejecutar el archivo *run.bat* para iniciar el servidor.  El archivo *run.bat* cuenta con las siguientes instrucciones (todo en una sola línea) :

	java -jar winstone-0.9.10.jar --webroot=root --httpPort=8089 
	     --commonLibFolder=lib --useJasper=true --useJNDI=true 
	     --jndi.resource.universidad=javax.sql.DataSource
	     --jndi.param.universidad.driverClassName=org.sqlite.JDBC 
	     --jndi.param.universidad.url=jdbc:sqlite:database/universidad.db
	
Luego se debe apuntar el visualizador (browser) de Web a la dirección http://localhost:8089/listaProfesores.jsp

*Nota: Es posible que el JDK de Java se encuentre instalado en otro directorio en su máquina.*

	
      
     