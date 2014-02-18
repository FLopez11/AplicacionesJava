Uso de JSP
==========

Una de las tecnologías más ampliamente utilizadas para el desarrollo de
aplicaciones Web bajo ambiente Java es JSP (Java Server Pages). Esta
herramienta permite crear una serie de plantillas HTML que incluyen
código incrustado (llamados *sniplets*) mediante marcadores especiales.

El uso de JSP simplifica la programación de *servlets* pues no es
necesario compilar código ya que esto se realiza de forma automática.
Además, debido a que la plantilla se escribe directamente en HTML es
factible utilizar editores y diseñadores de páginas Web para programar
la aplicación.

Creación de la base de datos
----------------------------

Para este ejemplo se creará una base de datos de prueba, llamada
*universidad.db*, utilizando *SQLite*. Es posible utilizar diferentes
herramientas para crear bases de datos en este formato, entre ellas se
encuentra el *plugin* para Firefox llamado *SQLite Manager*, o bien, la
herramienta *SQLiteBrowser*.

Por el momento solo se utilizará una tabla con información de profesores
de una universidad. El código SQL para crear dicha tabla sería el
siguiente:

::

    CREATE TABLE "profesor" (
      id INTEGER PRIMARY KEY ASC,  
      cedula VARCHAR,  
      nombre VARCHAR,  
      titulo VARCHAR,  
      area VARCHAR,  
      telefono VARCHAR
    );

Luego de crear la base de datos se agregaron algunos datos de ejemplo a
esta tabla. Las siguientes instrucciones SQL permiten poblar la tabla
alguna información:

::

    INSERT INTO profesor (id,cedula,nombre,titulo,area,telefono) 
      VALUES (1,'101110111','Carlos Perez Rojas','Licenciado',
                'Administracion','3456-7890');
    INSERT INTO profesor (id,cedula,nombre,titulo,area,telefono) 
      VALUES (2,'202220222','Luis Torres','Master',
                'Economia','6677-3456');
    INSERT INTO profesor (id,cedula,nombre,titulo,area,telefono) 
      VALUES (3,'303330333','Juan Castro','Licenciado',
                'Matematica','67455-7788');

Página de listado de profesores
-------------------------------

La primer página consistirá del listado de todos los profesores que se
encuentran en la base de datos. El código que se muestra a continuación
(llamado *listaProfesores.jsp*) establece la conexión con la base de
datos (utilizado *JDBC*), ejecuta la instrucción de consulta, recupera
los datos y crea una tabla HTML con la información obtenida.

::

    <%@ page import="java.sql.*" %>
    <%
       String driver="org.sqlite.JDBC";
       String url="jdbc:sqlite:database/universidad.db";
       Class.forName(driver);
       Connection conn=null;
       try {
         conn = DriverManager.getConnection(url);
         String sql="select * from profesor";
         Statement stmt = conn.createStatement();
         ResultSet rs = stmt.executeQuery(sql);
    %>
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
        <% while (rs.next()) {
           String id = rs.getString(1);
           String cedula = rs.getString(2);
           String nombre = rs.getString(3);
           String titulo = rs.getString(4);
         %>
           <tr><td><%= cedula %></td>
            <td><%= nombre %></td>
            <td><%= titulo %></td>
            <td><a href='/detalleProfesor.jsp?id=<%= id %>'>
                  <input type="submit" value="Detalle"/></a>
                <a href='/eliminarProfesor.jsp?id=<%= id %>'>
                  <input type="submit" value="Eliminar"/></a></td></tr>
        <% } %>
       </tbody>
       <tfoot>
        <tr><td><a href='/agregarProfesor.jsp'>
            <input type="submit" name="action" value="Agregar"/></a>
        </td><td></td><td></td><td></td></tr>
       </tfoot>
      </table>
    </html>
    <%
        rs.close();rs=null;
        stmt.close();stmt=null;
        if (conn!=null) 
          conn.close();
       } catch (Exception e) {
         e.printStackTrace();
       }
    %>

Es importante observar en este código que existen enlaces que accederán
a otras páginas para realizar acciones con los datos. Por ejemplo, el
enlace de "Detalle" ejecutará la página *detalleProfesor.jsp* con el
parámetro *ID*; y el enlace de "Eliminar" ejecutará la página
*eliminarProfesor.jsp* con el mismo parámetro *ID*. También está
presente otro enlace "Agregar" que invocará a la página
*agregarProfesor.jsp* pero sin parámetros.

Detalle del profesor
--------------------

La página *detalleProfesor.jsp* recibe como parámetro el *ID* de un
profesor, recupera sus datos desde la base y datos, y los muestra en un
formulario HTML. La estructura general de la consulta a la base de datos
es muy similar a la anterior con la diferencia que se recupera solo un
registro particular.

::

    <%@ page import="java.sql.*" %>
    <%
       String id = request.getParameter("id");
       String driver="org.sqlite.JDBC";
       String url="jdbc:sqlite:database/universidad.db";
       Class.forName(driver);
       Connection conn=null;
       try {
         conn = DriverManager.getConnection(url);
         String sql="select * from profesor where id='"+id+"'";
         Statement stmt = conn.createStatement();
         ResultSet rs = stmt.executeQuery(sql);
    %>
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
        <% rs.next();
           String cedula = rs.getString(2);
           String nombre = rs.getString(3);
           String titulo = rs.getString(4);
           String area = rs.getString(5);
           String telefono = rs.getString(6);
         %>
        <input type="hidden" name="id" value="<%= id %>"/>
        <tr><td>Nombre:</td><td>
          <input type="text" name="nombre" value="<%= nombre %>"/></td></tr>
        <tr><td>Cedula:</td><td>
          <input type="text" name="cedula" value="<%= cedula %>"/></td></tr>
        <tr><td>Titulo:</td><td>
          <input type="text" name="titulo" value="<%= titulo %>"/></td></tr>
        <tr><td>Area:</td><td>
          <input type="text" name="area" value="<%= area %>"/></td></tr>
        <tr><td>Telefono:</td><td>
          <input type="text" name="telefono" value="<%= telefono %>"/></td></tr>
        </tbody>
        <tfoot>
        <tr><td><input type="submit" value="Actualizar" /></td><td></td></tr>
        </tfoot>
       </tbody>
      </table>
      </form>
    </html>
    <%
        rs.close();rs=null;
        stmt.close();stmt=null;
        if (conn!=null) 
          conn.close();
       } catch (Exception e) {
         e.printStackTrace();
       }
    %>

Este código también cuenta con un enlace adicional "Actualizar" que
permite tomar la información del formulario y realizar la actualización
de datos en la base de datos, mediante la página
*actualizarProfesor.jsp*

Hoja de estilo
--------------

Con el fin de mejorar la apariencia de este ejemplo se ha desarrollado
una pequeña hoja de estilo (css) que permite organizar mejor los datos
de la tabla y el formulario. El archivo llamado *style.css* cuenta con
el siguiente código:

::

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

Ambiente de ejecución
---------------------

Para ejecutar este ejemplo es necesario contar con un *servidor de
servlets* que permita también la ejecución de plantillas *JSP*. La
herramienta más utilizada para esto es el *Apache Tomcat*, el cuál es
muy potente y cuenta con gran cantidad de parámetros de configuración.
Sin embargo, para propósito de desarrollo y depuración de programas
basta con un ambiente más liviano tal como *Winstone*.

*Winstone* consiste de un único archivo de menos de 350 KB, llamado
*winstone-0.9.10.jar*, el cual puede ser ejecutado directamente mediante
Java. Sin embargo, poder utilizar plantillas JSP se requiere de la
herramienta *Jasper* que consiste de múltiples librerías adicionales.

Para acceder a la base de datos *SQLite*, mediante *JDBC*, es necesario
contar con una librería que incluya el driver adecuado. Aún cuando
existen diferentes librerías que hacen esto, ninguna es pequeña.

Estructura de directorios
~~~~~~~~~~~~~~~~~~~~~~~~~

La ubicación de los diferentes archivos de código, y librerías se
muestra en el siguiente esquema de directorios:

::

    tutorial1
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
        juli-6.0.18.jar
        servlet-api-2.5.jar
        sqlite-jdbc-3.5.9.jar

Ejecución del ejemplo
---------------------

Teniendo instalado el JDK de Java (no el JRE) basta con ejecutar el
archivo *run.bat* para iniciar el servidor. El archivo *run.bat* cuenta
con las siguientes instrucciones:

::

    set PATH=C:\Java\jdk1.7.0\bin;%PATH%
    java -jar winstone-0.9.10.jar --httpPort=8089 ^ 
         --commonLibFolder=lib --useJasper=true --webroot=root

Luego se debe apuntar el visualizador (browser) de Web a la dirección
http://localhost:8089/listaProfesores.jsp

*Nota: Es posible que el JDK de Java se encuentre instalado en otro
directorio en su máquina.*
