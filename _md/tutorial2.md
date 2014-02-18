# Rutinas de Transacción

A continuación se presenta un ejemplo sencillo de la aplicación Web del Sistema Universitario, lo que servirá para mostrar el uso de diferentes capas y de cómo se debe estructurar una aplicación para obtener el mayor provecho de las aplicaciones empresariales.

En este ejemplo se combinarán tres patrones descritos por Fowler: *rutinas de transacción* para la lógica del dominio, *pasarela a fila de datos* para la capa de acceso a los datos, y *controlador frontal* para la capa de presentación. Más específicamente, las *rutinas de transacción* se estructurarán utilizando el patrón *comando*.

## Capa de acceso a datos

La capa de acceso a datos utilizará una *pasarela a filas de datos* que se conectará a una base de datos *SQLite*. Este tipo de técnica también requiere crear una clase que se encargue de crear la pasarela, llamada un "finder".

### Base de datos

Se utilizará una base de datos SQLite para administrar los datos. Dicha base de datos debe llevar por nombre *universidad.db* y debe estar ubicada en el directorio */tutorial2/database/*. El código SQL utilizado para generar la tabla de profesores sería el siguiente:

	CREATE TABLE profesor (id INTEGER PRIMARY KEY, cedula VARCHAR,
	  nombre VARCHAR, titulo VARCHAR, area VARCHAR, telefono VARCHAR);
	INSERT INTO profesor VALUES(1,'101110111','Carlos Perez','Licenciado',
	  'Administracion','3456-7890');
	INSERT INTO profesor VALUES(2,'202220222','Luis Torres','Master',
	  'Economia','6677-3456');
	INSERT INTO profesor VALUES(3,'303330333','Juan Castro','Licenciado',
	  'Matematica','6755-7788');

Para administrar una base de datos SQLite se puede utilizar alguno de los excelentes productos creados para ello, tal como *SQLiteman* ó el plugin para Firefox llamado *SQLite Manager*

### Pasarela a fila de datos

Para implementar la capa de acceso de datos se utiliza una *pasarela a filas de datos*. Para ello es necesario crear una clase que permita acceder a la información de la base de datos cuyo nombre es *ProfesorRowGateway.java*. Dicha clase residirá en el directorio */tutorial2/src/data/*

	package data;

	import java.util.*;
	import java.sql.*;
	import javax.sql.*;
	import org.springframework.jdbc.core.JdbcTemplate;

	public class ProfesorRowGateway {
		
	  private int id;
	  private String cedula;
	  private String nombre;
	  private String titulo;
	  private String area;
	  private String telefono;
	  private JdbcTemplate jdbcTemplate;
	  private DataSource dataSource;
		
	  ProfesorRowGateway() {};
		
	  ProfesorRowGateway(int id, String ced, String nomb,
									String tit, String area, String tel) {
			this.id=id; this.cedula=ced; this.nombre=nomb;
			this.titulo=tit;this.area=area;this.telefono=tel;
	  }
		
	  public void setId(int id) {this.id = id;}
	  public int getId() {return id;}
		
	  public void setCedula(String ced) {this.cedula=ced;}
	  public String getCedula() {return cedula;}
		
	  public void setNombre(String nomb) {this.nombre=nomb;}
	  public String getNombre() {return nombre;}
		
	  public void setTitulo(String tit) {this.titulo=tit;}
	  public String getTitulo() {return titulo;}
		
	  public void setArea(String area) {this.area=area;}
	  public String getArea() {return area;}
		
	  public void setTelefono(String tel) {this.telefono=tel;}
	  public String getTelefono() {return telefono;}

	  public void setDataSource(DataSource dataSource) {
			this.dataSource = dataSource;
	  }
		
	  private void createJdbcTemplate() {
			jdbcTemplate = new JdbcTemplate(dataSource);
	  }
  
	  private static final String insertStatement =
			"INSERT INTO profesor "+
			"VALUES (?,?,?,?,?,?)";
				
	  public int insert() {
			Random generator = new Random();
			int id = generator.nextInt();
			if (jdbcTemplate==null) createJdbcTemplate();
			  jdbcTemplate.update(insertStatement,
					 id,cedula,nombre,titulo,area,telefono);
			return id;
	  }
		
	  private static final String updateStatement =
			"UPDATE profesor "+
			"SET cedula = ?, nombre = ?, titulo = ?, "+
			"area = ?, telefono = ? WHERE id = ?";
		
	  public void update() {
			if (jdbcTemplate==null) createJdbcTemplate();
			  jdbcTemplate.update(updateStatement,
					  cedula,nombre,titulo,area,telefono,id);
	  }
  
	  private static final String deleteStatement =
			"DELETE FROM profesor "+
			"WHERE id = ?";
				
	  public void delete() {
			if (jdbcTemplate==null) createJdbcTemplate();
			  jdbcTemplate.update(deleteStatement,id);
	  }
		
	  public static ProfesorRowGateway load(DataSource ds, Map map) {
			ProfesorRowGateway prof=null;
			if (map==null) 
			  prof = new ProfesorRowGateway();
			else {
			  int id = ((Integer)map.get("id")).intValue();
			  String cedula = (String)map.get("cedula");
			  String nombre = (String)map.get("nombre");
			  String titulo = (String)map.get("titulo");
			  String area = (String)map.get("area");
			  String telefono = (String)map.get("telefono");
			  prof = new ProfesorRowGateway(id,cedula,nombre,titulo,area,telefono);
			}
			prof.setDataSource(ds);
			return prof;
	  }
	}


### Generación de objetos de pasarelas

Para manejar este tipo de técnica es necesario contar con otra clase encargada de crear las pasarelas. Esta nueva clase se define en el archivo *ProfesorFinder.java* y reside en el mismo directorio */tutorial2/src/data/*

	package data;

	import java.util.*;
	import java.sql.*;
	import javax.sql.*;
	import org.springframework.jdbc.core.JdbcTemplate;

	public class ProfesorFinder {
  
	  private JdbcTemplate jdbcTemplate;
	  private DataSource dataSource;
  
	  public void setDataSource(DataSource dataSource) {
			this.dataSource = dataSource;
			this.jdbcTemplate = new JdbcTemplate(dataSource);
	  }
  
	  public ProfesorRowGateway create() {
			return ProfesorRowGateway.load(dataSource,null);
	  }
  
	  private final static String findStatement =
			 "SELECT * "+
			 "FROM profesor "+
			 "WHERE id = ?";
		 
	  public ProfesorRowGateway find(String id) {
			List profs = 
			  jdbcTemplate.queryForList(findStatement,id);
			ProfesorRowGateway prof = 
			  ProfesorRowGateway.load(dataSource,(Map)profs.get(0));
			return prof;
	  }
  
	  private final static String findAllStatement =
			 "SELECT * "+
			 "FROM profesor ";
		 
	  public List<ProfesorRowGateway> findAll() {
			List result = new ArrayList();
			List profs = jdbcTemplate.queryForList(findAllStatement);
			for (int i=0; i<profs.size();i++)
			  result.add(ProfesorRowGateway.load(dataSource,(Map)profs.get(i)));
			return result;
	  }
	}

### Compilando la capa de datos

Se puede realizar la compilación de estas dos clases en forma separada del resto del código. Para ello es necesario contar con el framework *Spring 3* el cual se debe descargar y copiar todas las librerías *.jar* del directorio *lib* de dicho framework hacia el directorio */tutorial2/root/WEB-INF/lib/*. También es necesario contar en dicho directorio con el driver jdbc para *SQLite* y las librerías *commons* para manejo de conexiones a base de datos.

Específicamente las librerías que deben residir en el directorio */tutorial2/root/WEB-INF/lib/* son las siguientes:

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

La siguiente instrucción para ejecutar la compilación puede estar definida en un archivo *compileDataLayer.bat* residente en el directorio */tutorial2* (todo en una sola línea):

	javac -cp "root/WEB-INF/lib/*" -d root/WEB-INF/classes
		src/data/ProfesorRowGateway.java src/data/ProfesorFinder.java
    
Nota: La versión del JDK debe ser superior a 6.0

## Capa de presentación

El servicio de la universidad ha sido implementado como un *controlador frontal*, en donde cada rutina de transacción será implementada como un *comando*. El archivo del servicio se llamará *FrontController.java* y debe residir en el directorio */tutorial2/src/display/*

	package display;

	import java.io.*;
	import java.util.*;
	import javax.servlet.*;
	import javax.servlet.http.*;

	import org.springframework.web.context.*;
	import org.springframework.web.context.support.*;

	public class FrontController extends HttpServlet {
  
	  private WebApplicationContext context;
  
	  public void init(ServletConfig config) throws ServletException {
			super.init(config);
			context = 
			  WebApplicationContextUtils.getWebApplicationContext(getServletContext());
	  }
  
	  public void doGet(HttpServletRequest request,
									  HttpServletResponse response)
			throws ServletException,IOException {
			FrontCommand command = 
			   getCommand((String)request.getAttribute("command"));
			command.init(context,request,response);
			command.process();
	  }
  
	  private FrontCommand getCommand(String commandName) {
			try {
			  return (FrontCommand) getCommandClass(commandName).newInstance();
			} catch (Exception e) {
			  e.printStackTrace();
			}
			return null;
	  }
  
	  private Class getCommandClass(String commandName) {
			Class result;
			try {
			  result = Class.forName(commandName);
			} catch (ClassNotFoundException e) {
			  result = UnknownCommand.class;
			}
			return result;
	  }
	}

### La clase para los comandos

Como se indicó antes, cada transacción se implementa como un comando independiente de todos los demás. Existe una clase abstracta *FrontCommand.java* que permite definir los métodos comunes a cualquier comando. Este archivo también se ubica en el directorio */tutorial2/src/display/*

	package display;
	import java.util.*;

	import java.io.*;
	import javax.servlet.*;
	import javax.servlet.http.*;
	import org.springframework.web.context.*;

	public abstract class FrontCommand {

	  public WebApplicationContext context;
	  public HttpServletRequest request;
	  public HttpServletResponse response;
  
	  public void init(WebApplicationContext ctx, 
									   HttpServletRequest req, 
									   HttpServletResponse resp) {
			this.context = ctx;
			this.request = req;
			this.response = resp;
	  }
  
	  protected void forward(String target) 
			throws ServletException, IOException {
			RequestDispatcher dispatcher = 
			  context.getServletContext().getRequestDispatcher(target);
			dispatcher.forward(request,response);
	  }
  
	  public abstract void process() 
			throws ServletException, IOException;

	}

### Comandos desconocidos

Cuando se pretende ejecutar un comando desconocido es necesario interceptar dicha solicitud e imprimir un mensaje de advertencia. La clase *UnknownCommand.java* se encarga de esta tarea y reside en el mismo directorio */tutorial2/src/display/*

	package display;
	import java.io.*;
	import javax.servlet.*;

	public class UnknownCommand extends FrontCommand {
  
	  public void process() 
			throws ServletException, IOException {
			forward("/unknown.jsp");
	  } 
	}


También es necesario contar con una pequeña plantilla *JSP* que permite generar el mensaje que se presentará en pantalla. El código de esta plantilla se ubica en el archivo *unknown.jsp* que reside en el directorio */tutorial2/root/*

	<html>
	  <head>
			<title>Sistema Universitario</title>
	  </head>
	  <h1>Operación inválida</h1>
	</html>

### Compilando la capa de presentación

También, se puede realizar la compilación de estas clases de presentación en forma separada del resto del código de la aplicación. Para ello es necesario contar con las librerías del framework *Spring 3* (como se indicó antes) y con la librería *servlet-api.jar* ubicadas en el directorio */tutorial2/root/WEB-INF/lib/*. La siguiente instrucción para ejecutar la compilación puede estar definida en un archivo *compileDisplayLayer.bat* residente en el directorio */tutorial2/* (todo en una sola línea):

	javac -cp "root/WEB-INF/lib/*" -d root/WEB-INF/classes
		src/display/FrontController.java src/display/FrontCommand.java
		src/display/UnknownCommand.java

## La capa de lógica del dominio

Para implementar la capa de lógica del dominio se utilizará la técnica de *rutinas de transacción*. En dicho caso cada rutina es responsable de recuperar los parámetros de la consulta, acceder a la tabla de datos, procesar los datos, e invocar a la rutina de presentación.

### La rutina de listado

Para presentar el listado de profesores se crea la clase *ListaProfesores.java* en el directorio */tutorial2/src/domain/*

	package domain;

	import display.FrontCommand;
	import data.ProfesorRowGateway;
	import data.ProfesorFinder;

	import java.util.Map;
	import java.util.HashMap;
	import java.util.List;
	import java.util.ArrayList;
	import java.io.IOException;
	import java.sql.SQLException;
	import javax.servlet.ServletException;

	public class ListaProfesores extends FrontCommand {

	  public void process() 
			throws ServletException, IOException {
			  ProfesorFinder profs = 
			    (ProfesorFinder) context.getBean("profesorFinder");
			List<ProfesorRowGateway> data = profs.findAll();
			List param = new ArrayList();
			for (int i=0;i<data.size();i++) {
			  ProfesorRowGateway prof = data.get(i);
			  Map item = new HashMap();
			  item.put("id",prof.getId()+"");
			  item.put("cedula",prof.getCedula());
			  item.put("nombre",prof.getNombre());
			  item.put("titulo",prof.getTitulo());
			  item.put("area",prof.getArea());
			  item.put("telefono",prof.getTelefono());
			  param.add(item);
			}
			request.setAttribute("profesores",param);
			forward("/listaProfesores.jsp");
	  }
	}

#### Plantilla JSP

Adicionalmente se utilizará una *plantilla* JSP para realizar el formateo de página en código HTML. El archivo *listaProfesores.jsp* se encarga de esta tarea y residirá en el directorio */tutorial2/root/*

	<%@ page import="java.util.*" %>
	<html>
	  <head>
			<title>Sistema Universitario</title>
			<link rel="stylesheet" href="style.css">
	  </head>
	  <h1>Sistema Universitario</h1>
	  <h2>Listado de profesores</h2>
	  <% List profs = (List)request.getAttribute("profesores"); %>
	  <table>
	      <thead>
			<tr><th>Nombre</th><th>Título</th>
			    <th>Area</th><th>Acciones</th></tr>
			</thead>
			<tbody>
			<% for(int i = 0, n = profs.size(); i < n; i++) {
					Map prof = (Map) profs.get(i); %>
					<tr><td><%= prof.get("nombre") %></td>
					<td><%= prof.get("titulo") %></td>
					<td><%= prof.get("area") %></td>
					<td>
			<a href='/domain.DetalleProfesor?id=<%= prof.get("id") %>'>
				<input type="submit" value="Detalle"/></a>
			<a href='/domain.EliminarProfesor?id=<%= prof.get("id") %>'>
				<input type="submit" value="Eliminar"/></a></td></tr>
			<% } %>
			</tbody>
			<tfoot>
			  <tr><td><a href='/domain.AgregarProfesor'>
					<input type="submit" name="action" value="Agregar"/></a>
			  </td><td></td><td></td><td></td></tr>
			</tfoot>
	   </table>
	</html>

Nótese que la tabla generada cuenta con enlaces que invocarán la rutina que presenta el detalle del profesor (que se describe a continuación).

#### Hoja de estilo

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

### La rutina de detalle

La rutina de detalle de profesor presentará otra tabla HTML con la información detallada del profesor. Este archivo es llamado *DetalleProfesor.java* y se ubica en el mismo directorio */tutorial2/src/domain/*. Es importante observar la utilización del *id* del profesor para realizar la consulta a la pasarela de datos.

	package domain;

	import display.FrontCommand;
	import data.ProfesorFinder;
	import data.ProfesorRowGateway;

	import java.util.Map;
	import java.util.HashMap;
	import java.io.IOException;
	import javax.servlet.ServletException;

	public class DetalleProfesor extends FrontCommand {

	  public void process() 
			throws ServletException, IOException {
			  ProfesorFinder profs = 
					(ProfesorFinder) context.getBean("profesorFinder");
			  String id = request.getParameter("id");
			ProfesorRowGateway prof = profs.find(id);
			Map params = new HashMap();
			params.put("id",prof.getId()+"");
			params.put("cedula",prof.getCedula());
			params.put("nombre",prof.getNombre());
			params.put("titulo",prof.getTitulo());
			params.put("area",prof.getArea());
			params.put("telefono",prof.getTelefono());
			request.setAttribute("profesor",params);
			forward("/detalleProfesor.jsp");
	  }
	}


#### Plantilla JSP

La plantilla JSP que genera el código HTML se presenta a continuación. El código de la plantilla se define en un archivo llamado *detalleProfesor.jsp* que reside también en el directorio */tutorial2/root/*.

	<%@ page import="java.util.Map" %>
	<html>
	  <head>
			<title>Sistema Universitario</title>
			<link rel="stylesheet" href="style.css">
	  </head>
	  <h1>Sistema Universitario</h1>
	  <h2>Detalle de Profesor</h2>
	  <% Map prof = (Map)request.getAttribute("profesor"); %>
	  <form name="ActualizarProfesor" 
			action="/domain.ActualizarProfesor" method="get">
	  <input type="hidden" name="id" value="<%= prof.get("id") %>"/>
	  <table style="width:400px;">
	    <thead>
	      <tr><th></th><th></th></tr>
	    </thead>
	    <tbody>
			<tr><td>Nombre:</td><td><input type="text" name="nombre" 
					value="<%= prof.get("nombre") %>"/></td></tr>
			<tr><td>Cédula:</td><td><input type="text" name="cedula" 
					value="<%= prof.get("cedula") %>"/></td></tr>
			<tr><td>Título:</td><td><input type="text" name="titulo" 
					value="<%= prof.get("titulo") %>"/></td></tr>
			<tr><td>Area:</td><td><input type="text" name="area" 
					value="<%= prof.get("area") %>"/></td></tr>
			<tr><td>Teléfono:</td><td><input type="text" name="telefono" 
					value="<%= prof.get("telefono") %>"/></td></tr>
		</tbody>
	    <tfoot>
	      <tr><td><input type="submit" value="Actualizar" /></td><td></td></tr>
	    </tfoot>
	   </tbody>
	  </table>
	  </form>
	</html>

### Actualizar información

Se presenta también la rutina de transacción que permite actualizar los datos de un profesor. La lógica de esta rutina se ubica en el archivo *ActualizarProfesor.java* y reside en el directorio */tutorial2/src/domain/*.

	package domain;

	import display.FrontCommand;
	import data.ProfesorFinder;
	import data.ProfesorRowGateway;

	import java.util.Map;
	import java.util.HashMap;
	import java.io.IOException;
	import javax.servlet.ServletException;

	public class ActualizarProfesor extends FrontCommand {

	  public void process() 
			throws ServletException, IOException {
			  ProfesorFinder profs = 
					(ProfesorFinder) context.getBean("profesorFinder");
			  String id = request.getParameter("id");
			ProfesorRowGateway prof = profs.find(id);
			if (prof!=null) {
			  String cedula = request.getParameter("cedula");
			  if (cedula!=null) prof.setCedula(cedula);
			  String nombre = request.getParameter("nombre");
			  if (nombre!=null) prof.setNombre(nombre);
			  String titulo = request.getParameter("titulo");
			  if (titulo!=null) prof.setTitulo(titulo);
			  String area = request.getParameter("area");
			  if (area!=null) prof.setArea(area);
			  String telefono = request.getParameter("telefono");
			  if (telefono!=null) prof.setTelefono(telefono);
			  prof.update();
			}
			response.sendRedirect("domain.ListaProfesores");
	  }
	}


### Compilando la capa de dominio

Se puede realizar la compilación de estas clases de dominio en forma separada del resto del código de la aplicación. Para ello es necesario contar con las librerías del framework *Spring 3* (como se indicó antes) y con la librería *servlet-api.jar* ubicadas en el directorio */tutorial2/root/WEB-INF/lib/*. La siguiente instrucción para ejecutar la compilación puede estar definida en un archivo *compileDomainLayer.bat* residente en el directorio */tutorial2* (todo en una sola línea):

	javac -cp "root/WEB-INF/classes";"root/WEB-INF/lib/*"
		-d root/WEB-INF/classes src/domain/ListaProfesores.java
		src/domain/DetalleProfesor.java src/domain/ActualizarProfesor.java

## Configuración del contexto

El framework *Spring* permite crear archivos xml que definen la configuración del contexto de ejecución de la aplicación. El archivo de configuración llamado *context.xml* se deberá ubicar en el directorio */tutorial2/root/WEB-INF/* y contendrá la siguiente información.
  
	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
			xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
			xmlns:context="http://www.springframework.org/schema/context"
			xsi:schemaLocation="
					http://www.springframework.org/schema/beans
					http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
					http://www.springframework.org/schema/context
					http://www.springframework.org/schema/context/spring-context-3.0.xsd">  
			<bean id="profesorFinder" class="data.ProfesorFinder">
					<property name="dataSource" ref="dataSource"/>
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

Precisamente para configurar la fuente de datos se utilizará un archivo de propiedades llamado *jdbc.properties* y que residirá en el directorio */tutorial2/root/WEB-INF*. Su contenido es simplemente el siguiente:

	jdbc.driverClassName=org.sqlite.JDBC
	jdbc.url=jdbc:sqlite:database/universidad.db
	jdbc.username=sa
	jdbc.password=root

Sin embargo, note que una base de datos *SQLite* no requiere indicar el usuario y su clave.

## Configuración del servidor

El servidor de servlets requiere del archivo de configuración de la aplicación para conocer en donde se ubica la clase a ejecutar. Además este archivo permite indicar la ubicación y nombre del archivo de contexto. Estos archivos de configuración del servlet siempre se llaman *web.xml* y deben residir en el directorio */tutorial2/root/WEB-INF/*. Para este caso su contenido sería el siguiente:

	<?xml version="1.0" encoding="ISO-8859-1"?>
	<web-app xmlns="http://java.sun.com/xml/ns/j2ee"
	   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	   xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee
		http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd"
	   version="2.4">
	 <display-name>Sistema Universitario</display-name>
	 <description>Ejemplo de Rutinas de Transaccion</description>
	 <context-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>/WEB-INF/context.xml</param-value>
	 </context-param>
	 <listener>
			<listener-class>
			  org.springframework.web.context.ContextLoaderListener
			</listener-class>
	 </listener>
	 <filter>
			<filter-name>UrlRewriteFilter</filter-name>
			<filter-class>org.tuckey.web.filters.urlrewrite.UrlRewriteFilter</filter-class>
	 </filter>
	 <filter-mapping>
			<filter-name>UrlRewriteFilter</filter-name>
			<url-pattern>/*</url-pattern>
	 </filter-mapping>
	 <servlet>
	   <servlet-name>FrontController</servlet-name>
	   <servlet-class>display.FrontController</servlet-class>
	 </servlet>
	 <servlet-mapping>
	   <servlet-name>FrontController</servlet-name>
	   <url-pattern>/frontController</url-pattern>
	 </servlet-mapping>
	</web-app>

Como se puede observar en este archivo, se utilizará una librería especial que permite redireccionar las solicitudes que se hacen al servicio con base en su URL. Para esto es necesario contar con un archivo *urlrewrite.xml* que se muestra a continuación y que residirá en el mismo directorio */tutorial2/root/WEB-INF/*

	<?xml version="1.0" encoding="utf-8"?>
	<!DOCTYPE urlrewrite PUBLIC "-//tuckey.org//DTD UrlRewrite 2.6//EN"
	        "http://tuckey.org/res/dtds/urlrewrite2.6.dtd">
	<urlrewrite>
	  <rule>
	    <from>/*.css</from>
	    <to>.css</to>
	  </rule>
	  <rule>
	    <from>/(.*)$</from>
	    <to>/frontController</to>
	    <set name="command">$1</set>
	  </rule>
	</urlrewrite>

## Ejecución del tutorial

Este ejemplo se puede ejecutar bajo cualquier contenedor de Servlet. Por ejemplo, para realizar la ejecución de pruebas se puede utilizar un producto como el *Winstone Servlet Container* que permite ejecutar servlets de forma muy sencilla. Se debe descargar el programa *winstone-0.9.10.jar* y copiarlo en el directorio */tutorial2/*. Sin embargo, para lograr que *Winstone* ejecute plantillas *JSP* es necesario descargar algunas librerías adicionales que deben ser copiadas en el directorio */tutorial2/lib*:

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

Para ejecutar el servidor de servlets se puede crear un archivo de instrucciones, llamado *run.bat*, similar al siguiente en el directorio */tutorial2/* (todo en una sola línea):

	java -jar winstone-0.9.10.jar --httpPort=8089 --commonLibFolder=lib
	      --useJasper=true --webroot=root

Luego se puede acceder a la aplicación desde cualquier visualizador web y apuntando a la dirección http://localhost:8089/domain.ListaProfesores
