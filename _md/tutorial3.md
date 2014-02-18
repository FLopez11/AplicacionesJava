# Módulo de Tabla

A continuación se presenta el mismo ejemplo del tutorial anterior de una aplicación Web de un Sistema Universitario. En este ejemplo se combinarán otros tres patrones descritos por Fowler: *módulo de tabla* para la lógica del dominio, *pasarela a tabla de datos* para la capa de acceso a los datos, y *controlador de página* para la capa de presentación.

## Capa de acceso a datos

La capa de acceso a datos utilizará una *pasarela a tabla de datos* que se conectará a una base de datos *SQLite*.

### Base de datos

Se utilizará la misma base de datos SQLite del tutorial anterior para administrar los datos. Dicha base de datos debe llevar por nombre *universidad.db* y debe estar ubicada en el directorio */tutorial3/database/*. El código SQL utilizado para generar la tabla de profesores sería el siguiente:

	CREATE TABLE profesor (id INTEGER PRIMARY KEY, cedula VARCHAR, 
	  nombre VARCHAR, titulo VARCHAR, area VARCHAR, telefono VARCHAR);
	INSERT INTO profesor VALUES(1,'101110111','Carlos Perez',
	  'Licenciado','Administracion','3456-7890');
	INSERT INTO profesor VALUES(2,'202220222','Luis Torres',
	  'Master','Economia','6677-3456');
	INSERT INTO profesor VALUES(3,'303330333','Juan Castro',
	  'Licenciado','Matematica','6755-7788');

Para administrar una base de datos SQLite se puede utilizar alguno de los excelentes productos creados para ello, tal como *SQLiteman* ó el plugin para Firefox llamado *SQLite Manager*

### Pasarela a tabla de datos

Para implementar la capa de acceso de datos se utiliza una *pasarela a tabla de datos*. Para ello es necesario crear una clase abstracta que se encargue de almacenar la conexión jdbc. Esta clase se llamará *TableGateway.java* y residirá en el directorio */tutorial3/src/data/*.

	package data;

	import java.util.*;
	import javax.sql.*;
	import org.springframework.jdbc.core.JdbcTemplate;

	public abstract class TableGateway {

	  protected JdbcTemplate jdbcTemplate;

	  public void setDataSource(DataSource dataSource) {
		this.jdbcTemplate = new JdbcTemplate(dataSource);
	  }
	}

La otra clase necesaria es la que implementa la tabla de datos para los profesores. Este archivo se llama *ProfesorGateway.java* y reside en el mismo directorio */tutorial3/src/data/*.

	package data;

	import java.util.*;
	import javax.sql.*;
	import org.springframework.jdbc.core.JdbcTemplate;

	public class ProfesorGateway extends TableGateway {
  
	  private final static String findStatement =
		 "SELECT * "+
		 "FROM profesor "+
		 "WHERE id = ?";
	 
	  public Map<String, Object> find(String id) {
		List profs = jdbcTemplate.queryForList(findStatement,id);
		return (Map<String, Object>)profs.get(0);
	  }
  
	  private final static String findAllStatement =
		 "SELECT * "+
		 "FROM profesor ";
	 
	  public List<Map<String, Object>> findAll() {
		return jdbcTemplate.queryForList(findAllStatement);
	  }
  
	  private static final String insertStatement =
		"INSERT INTO profesor "+
	  "VALUES (?,?,?,?,?,?)";
	
	  public int insert(String cedula,String nombre,String titulo,
	    String area, String telefono) {
		Random generator = new Random();
		int id = generator.nextInt();
		  jdbcTemplate.update(insertStatement,
			 id,cedula,nombre,titulo,area,telefono);
		return id;
	  }
  
	  private static final String updateStatement =
		"UPDATE profesor "+
		"SET cedula = ?, nombre = ?, titulo = ?, "+
		"area = ?, telefono = ? WHERE id = ?";
  
	  public void update(int id,String cedula,String nombre,
	    String titulo, String area, String telefono) {
		  jdbcTemplate.update(updateStatement,
			  cedula,nombre,titulo,area,telefono,id);
	  }
  
	  private static final String deleteStatement =
		"DELETE FROM profesor "+
		"WHERE id = ?";
	
	  public void delete(int id) {
		  jdbcTemplate.update(deleteStatement,id);
	  }
	}

### Compilando la capa de datos

Se puede realizar la compilación de estas dos clases en forma separada del resto del código. Para ello es necesario contar con el framework *Spring 3* el cual se debe descargar y copiar todas las librerías *.jar* del directorio *lib* de dicho framework hacia el directorio */tutorial3/root/WEB-INF/lib/*. También es necesario contar en dicho directorio con el driver jdbc para *SQLite* y las librerías *commons* para manejo de conexiones a base de datos.

Específicamente las librerías que deben residir en el directorio */tutorial3/root/WEB-INF/lib/* son las siguientes:

* commons-dbcp-1.4.jar
* commons-logging-1.1.1.jar
* commons-pool-1.6.jar
* servlet-api.jar
* spring-asm-3.2.0.M1.jar
* spring-beans-3.2.0.M1.jar
* spring-context-3.2.0.M1.jar
* spring-core-3.2.0.M1.jar
* spring-expression-3.2.0.M1.jar
* spring-jdbc-3.2.0.M1.jar
* spring-tx.3.2.0.M1.jar
* spring-web-3.2.0.M1.jar
* sqlite-jdbc-3.5.9.jar

La siguiente instrucción para ejecutar la compilación puede estar definida en un archivo *compileData.bat* residente en el directorio */tutorial3* (todo en una sola línea):

	javac -cp "root/WEB-INF/classes";"root/WEB-INF/lib/*"
	  -d root/WEB-INF/classes src/data/TableGateway.java
	  src/data/ProfesorGateway.java

Nota: La versión del JDK debe ser superior a 6.0

## La capa de lógica del dominio

Para implementar la capa de lógica del dominio se utilizará la técnica de *módulo de tabla*. En este caso el módulo agrupa toda la lógica del dominio, pero no se encarga del acceso a datos. Para acceder a los datos se utiliza la *pasarela a tabla de datos* mostrada anteriormente.

La única clase necesaria sería la llamada *ProfesorModule.java* y residirá en el directorio */tutorial3/src/domain/*.

	package domain;

	import data.TableGateway;
	import data.ProfesorGateway;

	import java.util.Map;
	import java.util.List;
	import java.io.IOException;
	import javax.servlet.ServletException;

	public class ProfesorModule {
  
	  private ProfesorGateway gateway;
  
	  public void setGateway(TableGateway gateway) {
		this.gateway = (ProfesorGateway)gateway;
	  }
  
	  public void actualizar(int id, String cedula, String nombre, 
	    String titulo, String area, String telefono) throws Exception {
		if (id <= 0)
		  throw new Exception("Identificador de profesor incorrecto");
		if (titulo.toLowerCase().equals("bachiller") ||
			titulo.toLowerCase().equals("licenciado") ||
			titulo.toLowerCase().equals("master") ||
			titulo.toLowerCase().equals("doctor"))
		  gateway.update(id,cedula,nombre,titulo,area,telefono);
		else
		  throw new Exception("Error en título de profesor");
	  }
  
	  public Map<String,Object> buscar(int id) throws Exception {
		if (id <= 0)
		  throw new Exception("Identificador de profesor incorrecto");
		Map<String,Object> prof = gateway.find(id+"");
		return prof;
	  }
  
	  public List<Map<String,Object>> listado() throws Exception {
		List<Map<String,Object>> profs = gateway.findAll();
		return profs;
	  }
	}

### Compilando la capa de dominio

Para compilar la clase del dominio es necesario contar con las librerías del framework *Spring 3* (como se indicó antes). La siguiente instrucción para ejecutar la compilación puede estar definida en un archivo *compileDomain.bat* residente en el directorio */tutorial3/* (todo en una sola línea):


	javac -cp "root/WEB-INF/classes";"root/WEB-INF/lib/*"
	  -d root/WEB-INF/classes src/domain/ProfesorModule.java


## Capa de presentación

El servicio de la root ha sido implementado mediante *controladores de página*, en donde cada página se implementa como un controlador individual. La clase general para definir los controladores se llama *PageController.java* y debe residir en el directorio */tutorial3/src/display/*.

	package display;

	import java.io.*;
	import java.util.*;
	import javax.servlet.*;
	import javax.servlet.http.*;

	import org.springframework.web.context.*;
	import org.springframework.web.context.support.*;

	public class PageController extends HttpServlet {
  
	  protected WebApplicationContext context;
  
	  public void init(ServletConfig config) throws ServletException {
		super.init(config);
		context = 
		  WebApplicationContextUtils.getWebApplicationContext(getServletContext());
	  }
  
	   protected void forward(String target, HttpServletRequest request,
						HttpServletResponse response) 
		throws ServletException, IOException {
		RequestDispatcher dispatcher = 
		  context.getServletContext().getRequestDispatcher(target);
		dispatcher.forward(request,response);
	  }
	}


### El controlador de listado de profesores

El primer controlador de página es el que permite mostrar el listado de profesores. Este archivo se llama *ListaProfesores.java* y reside en el mismo directorio */tutorial3/src/display/*.


	package display;
	import java.util.*;
	import java.io.*;
	import javax.servlet.*;
	import javax.servlet.http.*;
	import org.springframework.web.context.*;

	import domain.ProfesorModule;

	public class ListaProfesores extends PageController {
  
	  public void doGet(HttpServletRequest request,
						HttpServletResponse response)
		throws ServletException, IOException {
		ProfesorModule module = 
		  (ProfesorModule) context.getBean("profesorModule");
		try {
		  List data = module.listado();
		  request.setAttribute("profesores",data);
		  forward("/listaProfesores.jsp",request,response);
		} catch (Exception e) {
		  request.setAttribute("mensaje",e.getMessage());
		  forward("/paginaError.jsp",request,response);
		}
	  }
	}

#### La plantilla JSP

Adicionalmente se utilizará, con en el tutorial anterior, una *plantilla* JSP para realizar el formateo de página en código HTML. El archivo *listaProfesores.jsp* se encarga de esta tarea y residirá en el directorio */tutorial3/root/*.

	<%@ page import="java.util.*" %>
	<html>
	  <head>
	    <title>Sistema Universitario</title>
	    <link rel="stylesheet" href="style.css">
		<meta http-equiv="Content-Type" content="text/html; 
		      charset=UTF-8" />
	  </head>
	  <h1>Sistema Universitario</h1>
	  <h2>Listado de profesores</h2>
	  <% List profs = (List)request.getAttribute("profesores"); %>
	  <table>
	    <thead>
	    <tr><th>Nombre</th><th>T&iacute;tulo</th><th>Area</th>
	        <th>Acciones</th></tr>
	    </thead>
	    <tbody>
	    <% for(int i = 0, n = profs.size(); i < n; i++) {
	         Map prof = (Map) profs.get(i); %>
	        <tr><td><%= prof.get("nombre") %></td>
	        <td><%= prof.get("titulo") %></td>
	        <td><%= prof.get("area") %></td>
	        <td><a href='/detalleProfesor?id=<%= prof.get("id") %>'>
	              <input type="submit" value="Detalle"/></a>
	            <a href='/eliminarProfesor?id=<%= prof.get("id") %>'>
	              <input type="submit" value="Eliminar"/></a></td></tr>
	    <% } %>
	  </tbody>
	    <tfoot>
	      <tr><td><a href='/agregarProfesor'>
	        <input type="submit" name="action" value="Agregar"/></a>
	      </td><td></td><td></td><td></td></tr>
	    </tfoot>
	  </table>
	</html>

Nótese que la tabla generada cuenta con enlaces que invocarán la rutina que presenta el detalle del profesor (que se describe a continuación).

#### Hoja de estilo

Con el fin de mejorar la apariencia de este ejemplo se ha desarrollado una pequeña hoja de estilo (css) que permite organizar mejor los datos de la tabla y el formulario. El archivo llamado *style.css* cuenta con el siguiente código y y reside en el directorio */tutorial3/root/*.:

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

### El controlador de detalle de profesor

El controlador de detalle de profesor presentará otra tabla HTML con la información detallada del profesor. Este controlador es llamado *DetalleProfesor.java* y se ubica en el mismo directorio */tutorial3/src/display/*. Es importante observar la utilización del *id* del profesor para realizar la consulta al módula de tabla.

	package display;
	import java.util.*;
	import java.io.*;
	import javax.servlet.*;
	import javax.servlet.http.*;
	import org.springframework.web.context.*;

	import domain.ProfesorModule;

	public class DetalleProfesor extends PageController {
  
	  public void doGet(HttpServletRequest request,
						HttpServletResponse response)
		throws ServletException, IOException {
		ProfesorModule module = 
		  (ProfesorModule) context.getBean("profesorModule");
		try {
		  String id = request.getParameter("id");
		  int idProf = Integer.parseInt(id); 
		  Map prof = module.buscar(idProf);
		  request.setAttribute("profesor",prof);
		  forward("/detalleProfesor.jsp",request,response);
		} catch (Exception e) {
		  request.setAttribute("mensaje",e.getMessage());
		  forward("/paginaError.jsp",request,response);
		}
	  }
	}


#### Plantilla JSP

La plantilla JSP que genera el código HTML del detalle del profesor, se presenta a continuación. El código de la plantilla se define en un archivo llamado *detalleProfesor.jsp* que reside también en el directorio */tutorial3/root/*.

	<html>
	  <head>
	    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8"/>
	    <title>Sistema Universitario</title>
	     <link rel="stylesheet" href="style.css">
	  </head>
	  <h1>Sistema Universitario</h1>
	  <h2>Detalle de Profesor</h2>
	  <% Map prof = (Map)request.getAttribute("profesor"); %>
	  <form name="ActualizarProfesor" action="/actualizarProfesor" method="get">
	  <input type="hidden" name="id" value="<%= prof.get("id") %>"/>
	  <table style="width:400px;">
	    <thead>
	      <tr><th></th><th></th></tr>
	    </thead>
	    <tbody>
	    <tr><td>Nombre:</td><td><input type="text" name="nombre" 
	        value="<%= prof.get("nombre") %>"/></td></tr>
	    <tr><td>C&eacute;dula:</td><td><input type="text" name="cedula" 
	        value="<%= prof.get("cedula") %>"/></td></tr>
	    <tr><td>T&iacute;tulo:</td><td><input type="text" name="titulo" 
	        value="<%= prof.get("titulo") %>"/></td></tr>
	    <tr><td>Area:</td><td><input type="text" name="area" 
	        value="<%= prof.get("area") %>"/></td></tr>
	    <tr><td>Tel&eacute;fono:</td><td><input type="text" name="telefono" 
	        value="<%= prof.get("telefono") %>"/></td></tr>
	    </tbody>
	    <tfoot>
	    <tr><td><input type="submit" value="Actualizar" />
	        </td><td></td></tr>
	    </tfoot>
	  </table>
	  </form>
	</html>

### El controlador para actualizar información

Se presenta también el controlador de página que permite actualizar los datos de un profesor. La lógica de este controlador se ubica en el archivo *ActualizarProfesor.java* y reside en el directorio */tutorial3/src/display/*.

	package display;
	import java.util.*;
	import java.io.*;
	import javax.servlet.*;
	import javax.servlet.http.*;
	import org.springframework.web.context.*;

	import domain.ProfesorModule;

	public class ActualizarProfesor extends PageController {
  
	  public void doGet(HttpServletRequest request,
						HttpServletResponse response)
		throws ServletException, IOException {
		ProfesorModule module = 
		  (ProfesorModule) context.getBean("profesorModule");
		try {
		  String id = request.getParameter("id");
		  int idProf = Integer.parseInt(id);
		  String cedula = request.getParameter("cedula");
		  String nombre = request.getParameter("nombre");
		  String titulo = request.getParameter("titulo");
		  String area = request.getParameter("area");
		  String telefono = request.getParameter("telefono");
		  module.actualizar(idProf,cedula,nombre,titulo,area,telefono);
		  response.sendRedirect("listaProfesores");
		} catch (Exception e) {
		  request.setAttribute("mensaje",e.getMessage());
		  forward("/paginaError.jsp",request,response);
		}
	  }
	}

### Compilando la capa de presentación

Para compilar la capa de presentación es necesario contar con las librerías del framework *Spring 3* (como se indicó antes) y con la librería *servlet-api.jar* ubicadas en el directorio */tutorial3/root/WEB-INF/lib/*. La siguiente instrucción para ejecutar la compilación puede estar definida en un archivo *compileDisplay.bat* residente en el directorio */tutorial3/* (todo en una sola línea):

	javac -cp "root/WEB-INF/classes";"root/WEB-INF/lib/*"
	  -d root/WEB-INF/classes src/display/PageController.java
	  src/display/ListaProfesores.java src/display/ActualizarProfesor.java
	  src/display/DetalleProfesor.java


## Configuración del contexto

El framework *Spring* permite crear archivos xml que definen la configuración del contexto de ejecución de la aplicación. El archivo de configuración llamado *context.xml* se deberá ubicar en el directorio */tutorial3/root/WEB-INF/* y contendrá la siguiente información.

	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xmlns:context="http://www.springframework.org/schema/context"
		xsi:schemaLocation="
			http://www.springframework.org/schema/beans
			http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
			http://www.springframework.org/schema/context
			http://www.springframework.org/schema/context/spring-context-3.0.xsd">  
		<bean id="profesorGateway" class="data.ProfesorGateway">
			<property name="dataSource" ref="dataSource"/>
		</bean>   
		 <bean id="profesorModule" class="domain.ProfesorModule">
			<property name="gateway" ref="profesorGateway"/>
		</bean>   
		<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"
		   destroy-method="close">
			<property name="driverClassName" value="${jdbc.driverClassName}"/>
			<property name="url" value="${jdbc.url}"/>
			<property name="username" value="${jdbc.username}"/>
			<property name="password" value="${jdbc.password}"/>
		</bean>
		<context:property-placeholder location="WEB-INF/jdbc.properties"/>
	</beans>

Los aspectos importantes que se pueden observar en este archivo son la declaración de una instancia (singleton) al constructor de pasarelas y la declaración de la fuente de datos *JDBC*.

Precisamente para configurar la fuente de datos se utilizará un archivo de propiedades llamado *jdbc.properties* y que residirá en el directorio */tutorial3/root/WEB-INF*. Su contenido es simplemente el siguiente:

	jdbc.driverClassName=org.sqlite.JDBC
	jdbc.url=jdbc:sqlite:database/universidad.db
	jdbc.username=sa
	jdbc.password=root

Sin embargo, note que una base de datos *SQLite* no requiere indicar el usuario y su clave.

## Configuración del servidor

El servidor de servlets requiere del archivo de configuración de la aplicación para conocer en donde se ubica la clase a ejecutar. Además este archivo permite indicar la ubicación y nombre del archivo de contexto. Estos archivos de configuración del servlet siempre se llaman *web.xml* y deben residir en el directorio */tutorial3/root/WEB-INF/*. Para este caso su contenido sería el siguiente:

	<?xml version="1.0" encoding="ISO-8859-1"?>
	<web-app xmlns="http://java.sun.com/xml/ns/j2ee"
	   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	   xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee
						   http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd"
	   version="2.4">
	 <display-name>Sistema Universitario</display-name>
	 <description>Ejemplo de Módulo de Tabla</description>
	 <context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>/WEB-INF/context.xml</param-value>
	 </context-param>
	 <listener>
		<listener-class>
		  org.springframework.web.context.ContextLoaderListener
		</listener-class>
	 </listener>
	 <servlet>
	   <servlet-name>ActualizarProfesor</servlet-name>
	   <servlet-class>display.ActualizarProfesor</servlet-class>
	 </servlet>
	 <servlet>
	   <servlet-name>DetalleProfesor</servlet-name>
	   <servlet-class>display.DetalleProfesor</servlet-class>
	 </servlet>
	 <servlet>
	   <servlet-name>ListaProfesores</servlet-name>
	   <servlet-class>display.ListaProfesores</servlet-class>
	 </servlet>
	 <servlet-mapping>
	   <servlet-name>ActualizarProfesor</servlet-name>
	   <url-pattern>/actualizarProfesor</url-pattern>
	 </servlet-mapping>
	 <servlet-mapping>
	   <servlet-name>DetalleProfesor</servlet-name>
	   <url-pattern>/detalleProfesor</url-pattern>
	 </servlet-mapping>
	 <servlet-mapping>
	   <servlet-name>ListaProfesores</servlet-name>
	   <url-pattern>/listaProfesores</url-pattern>
	 </servlet-mapping>
	</web-app>

## Ejecución del tutorial

Este ejemplo se puede ejecutar bajo cualquier contenedor de Servlet. Por ejemplo, para realizar la ejecución de pruebas se puede utilizar un producto como el *Winstone Servlet Container* que permite ejecutar servlets de forma muy sencilla. Se debe descargar el programa *winstone-0.9.10.jar* y copiarlo en el directorio */tutorial3/*. Sin embargo, para lograr que *Winstone* ejecute plantillas *JSP* es necesario descargar algunas librerías adicionales que deben ser copiadas en el directorio */tutorial3/lib*:

* el-api-6.0.18.jar
* jasper-6.0.18.jar
* jasper-el-6.0.18.jar
* jasper-jdt-6.0.18.jar
* jsp-api-6.0.18.jar
* jstl-api-1.2.jar
* jstl-impl-1.2.jar
* jtds-1.2.4.jar
* juli-6.0.18.jar
* servlet-api-2.5.jar
* servlet-api.jar
* urlrewrite-3.2.0.jar

Para ejecutar el servidor de servlets se puede crear un archivo de instrucciones, llamado *run.bat*, similar al siguiente en el directorio */tutorial3/*:

	java -jar winstone-0.9.10.jar --httpPort=8089 --commonLibFolder=lib 
	  --useJasper=true --webroot=root

Luego se puede acceder a la aplicación desde cualquier visualizador web y apuntando a la dirección http://localhost:8089/listaProfesores