Modelo del Dominio
==================

A continuación se presenta el ejemplo de la aplicación Web de un Sistema
Universitario pero en este caso utilizando un *modelo del Dominio*.
Específicamente, en este ejemplo se combinarán otros patrones descritos
por Fowler: *modelo del dominio* para la lógica del dominio, y
*controlador de página* para la capa de presentación.

La capa de lógica del dominio
-----------------------------

Para implementar la capa de lógica del dominio se utilizará la técnica
de *modelo del dominio*. En este caso el modelo agrupa toda la lógica
del dominio, pero no se encarga del acceso a datos.

La primer clase necesaria consiste en la entidad profesor
*Profesor.java* y residirá en el directorio */tutorial4/src/domain/*.
Esta clase es la que contendría la lógica del dominio.

::

    package domain;

    public class Profesor {
      private int id;
      private String cedula;
      private String nombre;
      private String titulo;
      private String area;
      private String telefono;

      public Profesor () {};
      public void setId(int id) {this.id=id;}
      public void setCedula(String cedula) {this.cedula=cedula;}
      public void setNombre(String nombre) {this.nombre=nombre;}
      public void setTitulo(String titulo) throws Exception {
        if (titulo.toLowerCase().equals("bachiller") ||
            titulo.toLowerCase().equals("licenciado") ||
            titulo.toLowerCase().equals("master") ||
            titulo.toLowerCase().equals("doctor"))
          this.titulo=titulo;
        else
          throw new Exception("Error en título de profesor");
      }
      public void setArea(String area) {this.area=area;}
      public void setTelefono(String telefono) {this.telefono=telefono;}

      public int getId() {return id;}
      public String getCedula() {return cedula;}
      public String getNombre() {return nombre;}
      public String getTitulo() {return titulo;}
      public String getArea() {return area;}
      public String getTelefono() {return telefono;}
    }

El repositorio de datos
~~~~~~~~~~~~~~~~~~~~~~~

Para mantener almacenados y recuperar los diferentes objetos, se
utilizará un objeto llamado *ProfesorRepository.java* residente en el
mismo directorio */tutorial4/src/domain/*. Esta clase mantendrá los
datos de los objetos en memoria. Por tanto no será necesario utilizar
una base de datos en este tutorial.

::

    package domain;

    import java.util.Map;
    import java.util.HashMap;
    import java.util.Collection;

    public class ProfesorRepository {
      private Map<String,Profesor> profesores;

      ProfesorRepository() {
        profesores = new HashMap<String,Profesor>();
      }
      public boolean insertProfesor(Profesor prof) {
        if (profesores.containsKey(prof.getId()))
          return false;
        else
          profesores.put(prof.getId()+"",prof);
        return true;
      }
      public boolean deleteProfesor(Profesor prof) {
        if (!profesores.containsKey(prof.getId()))
          return false;
        else
          profesores.remove(prof.getId());
        return true;
      }
      public Profesor findProfesor(String id) {
        if (!profesores.containsKey(id))
          return null;
        else
          return profesores.get(id);
      }
      public boolean updateProfesor(Profesor prof) {
        if (!profesores.containsKey(prof.getId()+""))
          return false;
        else
          profesores.put(prof.getId()+"",prof);
        return true;
      }
      public Collection findAllProfesor() {
        return profesores.values();
      }
      public void setProfesores(Map profesores) {
        this.profesores = profesores;
      }
    }

La fábrica de objetos
~~~~~~~~~~~~~~~~~~~~~

Generalmente cuando se elabora un modelo del dominio es importante crear
una clase aparte que se encargue de crear instancias de objetos. En este
caso se utilizará la clase *ProfesorFactory.java* y se ubicará en el
mismo directorio */tutorial4/src/domain/*

::

    package domain;

    public class ProfesorFactory {
      public Profesor Create(int id,String cedula,String nombre,
                      String titulo,String area,String telefono) {
        try {
          Profesor prof = new Profesor();
          prof.setId(id);
          prof.setCedula(cedula);
          prof.setNombre(nombre);
          prof.setTitulo(titulo);
          prof.setArea(area);
          prof.setTelefono(telefono);    
          return prof;
        } catch (Exception e) {
          return null;
        }
      }
    }

Compilando la capa del dominio
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Se puede realizar la compilación de estas tres clases en forma separada
del resto del código. La siguiente instrucción para ejecutar la
compilación puede estar definida en un archivo *compileDomainLayer.bat*
residente en el directorio */tutorial4/* (todo en una sola línea):

::

    javac -d root/WEB-INF/classes src/domain/Profesor.java
      src/domain/ProfesorFactory.java src/domain/ProfesorRepository.java

Nota: La versión del JDK debe ser superior a 6.0

Capa de presentación
--------------------

El servicio de la universidad ha será implementado mediante
*controladores de página*, en donde cada página se implementa como un
controlador individual. La clase general para definir los controladores
se llama *PageController.java* y debe residir en el directorio
*/tutorial4/src/display/*.

::

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

El controlador de listado de profesores
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

El primer controlador de página es el que permite mostrar el listado de
profesores. Este archivo se llama *ListaProfesores.java* y reside en el
mismo directorio */tutorial4/src/display/*.

::

    package display;
    import java.util.*;
    import java.io.*;
    import javax.servlet.*;
    import javax.servlet.http.*;
    import org.springframework.web.context.*;

    import domain.ProfesorRepository;
    import domain.Profesor;

    public class ListaProfesores extends PageController {

      public void doGet(HttpServletRequest request,
                        HttpServletResponse response)
        throws ServletException, IOException {
          ProfesorRepository profesores = 
            (ProfesorRepository) context.getBean("profesorRepository");
        try {
                Collection lista = profesores.findAllProfesor();
                List data = new ArrayList();
                Iterator itr = lista.iterator();
                while (itr.hasNext()) {
                    Profesor prof = (Profesor)itr.next();
                    ProfesorDTO dto = ProfesorAssembler.Create(prof);
                    data.add(dto);
                }
          request.setAttribute("profesores",data);
          forward("/listaProfesores.jsp",request,response);
            } catch (Exception e) {
                request.setAttribute("mensaje",e.getMessage());
                forward("/paginaError.jsp",request,response);
            }
      }
    }

La plantilla JSP
^^^^^^^^^^^^^^^^

Adicionalmente se utilizará, con en el tutorial anterior, una
*plantilla* JSP para realizar el formateo de página en código HTML. El
archivo *listaProfesores.jsp* se encarga de esta tarea y residirá en el
directorio */tutorial4/root/*.

::

    <%@ page import="java.util.*" %>
    <%@ page import="display.*" %>
    <html>
      <head>
        <title>Sistema Universitario</title>
        <link rel="stylesheet" href="style.css">
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
      </head>
      <h1>Sistema Universitario</h1>
      <h2>Listado de profesores</h2>
      <% List profs = (List)request.getAttribute("profesores"); %>
      <table>
        <thead>
        <tr><th>Nombre</th><th>T&iacute;tulo</th><th>Area</th><th>Acciones</th></tr>
        </thead>
        <tbody>
        <% for(int i = 0, n = profs.size(); i < n; i++) {
             ProfesorDTO prof = (ProfesorDTO) profs.get(i); %>
            <tr><td><%= prof.nombre %></td>
            <td><%= prof.titulo %></td>
            <td><%= prof.area %></td>
            <td><a href='/detalleProfesor?id=<%= prof.id %>'>
                  <input type="submit" value="Detalle"/></a>
                <a href='/eliminarProfesor?id=<%= prof.id %>'>
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

Nótese que la tabla generada cuenta con enlaces que invocarán la rutina
que presenta el detalle del profesor (que se describe a continuación).

Hoja de estilo
^^^^^^^^^^^^^^

Con el fin de mejorar la apariencia de este ejemplo se ha desarrollado
una pequeña hoja de estilo (css) que permite organizar mejor los datos
de la tabla y el formulario. El archivo llamado *style.css* cuenta con
el siguiente código y y reside en el directorio */tutorial4/root/*.:

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

El controlador de detalle de profesor
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

El controlador de detalle de profesor presentará otra tabla HTML con la
información detallada del profesor. Este controlador es llamado
*DetalleProfesor.java* y se ubica en el mismo directorio
*/tutorial4/src/display*. Es importante observar la utilización del *id*
del profesor para realizar la consulta al módula de tabla.

::

    package display;
    import java.util.*;
    import java.io.*;
    import javax.servlet.*;
    import javax.servlet.http.*;
    import org.springframework.web.context.*;

    import domain.ProfesorRepository;
    import domain.Profesor;

    public class DetalleProfesor extends PageController {

      public void doGet(HttpServletRequest request,
                        HttpServletResponse response)
        throws ServletException, IOException {
          ProfesorRepository profesores =
              (ProfesorRepository) context.getBean("profesorRepository");
        try {
                String id = request.getParameter("id");
                int idProf = Integer.parseInt(id); 
                Profesor prof = profesores.findProfesor(idProf+"");
                ProfesorDTO dto = ProfesorAssembler.Create(prof);
          request.setAttribute("profesor",dto);
          forward("/detalleProfesor.jsp",request,response);
            } catch (Exception e) {
                request.setAttribute("mensaje",e.getMessage());
                forward("/paginaError.jsp",request,response);
            }
      }
    }

Plantilla JSP
^^^^^^^^^^^^^

La plantilla JSP que genera el código HTML del detalle del profesor, se
presenta a continuación. El código de la plantilla se define en un
archivo llamado *detalleProfesor.jsp* que reside también en el
directorio */tutorial4/root/*.

::

    <%@ page import="java.util.Map" %>
    <%@ page import="display.*" %>
    <html>
      <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8"/>
        <title>Sistema Universitario</title>
         <link rel="stylesheet" href="style.css">
      </head>
      <h1>Sistema Universitario</h1>
      <h2>Detalle de Profesor</h2>
      <% ProfesorDTO prof = (ProfesorDTO)request.getAttribute("profesor"); %>
      <form name="ActualizarProfesor" action="/actualizarProfesor" method="get">
      <input type="hidden" name="id" value="<%= prof.id %>"/>
      <table>
        <thead>
          <tr><th></th><th></th></tr>
        </thead>
        <tbody>
        <tr><td>Nombre:</td><td><input type="text" name="nombre" 
           value="<%= prof.nombre %>"/></td></tr>
        <tr><td>C&eacute;dula:</td><td><input type="text" name="cedula"
           value="<%= prof.cedula %>"/></td></tr>
        <tr><td>T&iacute;tulo:</td><td><input type="text" name="titulo"
           value="<%= prof.titulo %>"/></td></tr>
        <tr><td>Area:</td><td><input type="text" name="area"
           value="<%= prof.area %>"/></td></tr>
        <tr><td>Tel&eacute;fono:</td><td><input type="text" name="telefono" 
           value="<%= prof.telefono %>"/></td></tr>
        </tbody>
        <tfoot>
        <tr><td><input type="submit" value="Actualizar" /></td><td></td></tr>
        </tfoot>
      </table>
      </form>
    </html>

El controlador para actualizar información
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Se presenta también el controlador de página que permite actualizar los
datos de un profesor. La lógica de este controlador se ubica en el
archivo *ActualizarProfesor.java* y reside en el directorio
*/tutorial4/src/display/*.

::

    package display;
    import java.util.*;
    import java.io.*;
    import javax.servlet.*;
    import javax.servlet.http.*;
    import org.springframework.web.context.*;

    import domain.ProfesorRepository;
    import domain.Profesor;

    public class ActualizarProfesor extends PageController {

      public void doGet(HttpServletRequest request,
                        HttpServletResponse response)
        throws ServletException, IOException {
          ProfesorRepository profesores = 
            (ProfesorRepository) context.getBean("profesorRepository");
        try {
                String id = request.getParameter("id");
                int idProf = Integer.parseInt(id);
                String cedula = request.getParameter("cedula");
                String nombre = request.getParameter("nombre");
                String titulo = request.getParameter("titulo");
                String area = request.getParameter("area");
                String telefono = request.getParameter("telefono");
                Profesor prof = profesores.findProfesor(idProf+"");
                try {
                    if (cedula!=null) prof.setCedula(cedula);
                    if (nombre!=null) prof.setNombre(nombre);
                    if (titulo!=null) prof.setTitulo(titulo);
                    if (area!=null) prof.setArea(area);
                    if (telefono!=null) prof.setTelefono(telefono);
                } catch (Exception e) {}
          response.sendRedirect("listaProfesores");
            } catch (Exception e) {
                request.setAttribute("mensaje",e.getMessage());
                forward("/paginaError.jsp",request,response);
            }
      }
    }

El DTO de profesor
~~~~~~~~~~~~~~~~~~

En esta implementación se utiliza una clase tipo DTO (Data Transfer
Object) que facilite el paso de información hacia las vistas de datos.
Para ello se utiliza la clase *ProfesorDTO.java* residente en el
directorio */tutorial4/src/display/*.

::

    package display;

    public class ProfesorDTO {
      public int id;
      public String cedula;
      public String nombre;
      public String titulo;
      public String area;
      public String telefono;
    }

El ensamblador del DTO
^^^^^^^^^^^^^^^^^^^^^^

Adicionalmente es necesario contar con una clase que realice el
ensamblaje del DTO a partir de la entidad de profesor. Aquí se utiliza
la clase *ProfesorAssembler.java* residente en el mismo directorio
*/tutorial4/src/display/*.

::

    package display;

    import domain.Profesor;

    public class ProfesorAssembler {
      public static ProfesorDTO Create(Profesor prof) {
        ProfesorDTO dto = new ProfesorDTO();
        dto.id = prof.getId();
        dto.cedula = prof.getCedula();
        dto.nombre = prof.getNombre();
        dto.titulo = prof.getTitulo();
        dto.area = prof.getArea();
        dto.telefono = prof.getTelefono();
        return dto;
      }
    }

Compilando la capa de presentación
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Para compilar la capa de presentación es necesario contar con las
librerías del framework *Spring 3* (como se indicó antes) y con la
librería \*servlet-api.jar\*\* ubicadas en el directorio
*/tutorial4/root/WEB-INF/lib/*.

Específicamente las librerías necesarias son las siguientes:

-  servlet-api.jar
-  spring-asm-3.2.0.M1.jar
-  spring-beans-3.2.0.M1.jar
-  spring-context-3.2.0.M1.jar
-  spring-core-3.2.0.M1.jar
-  spring-expression-3.2.0.M1.jar
-  spring-jdbc-3.2.0.M1.jar
-  spring-tx.3.2.0.M1.jar
-  spring-web-3.2.0.M1.jar

La siguiente instrucción para ejecutar la compilación puede estar
definida en un archivo *compileDisplayLayer.bat* residente en el
directorio */tutorial4/* (todo en una sola línea):

::

    javac -cp "root/WEB-INF/classes";"root/WEB-INF/lib/*"
      -d root/WEB-INF/classes src/display/PageController.java
      src/display/ActualizarProfesor.java src/display/DetalleProfesor.java
      src/display/ListaProfesores.java src/display/ProfesorAssembler.java
      src/display/ProfesorDTO.java

Configuración del contexto
--------------------------

El framework *Spring* permite crear archivos *xml* que definen la
configuración del contexto de ejecución de la aplicación. El archivo de
configuración llamado *context.xml* se deberá ubicar en el directorio
*/tutorial4/root/WEB-INF/* y contendrá la siguiente información.

::

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xmlns:context="http://www.springframework.org/schema/context"
      xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context-3.0.xsd">
      <bean id="profesorRepository" class="domain.ProfesorRepository">
        <property name="profesores">
          <map>
            <entry>
              <key><value>1</value></key>
              <ref bean="P001" />
            </entry>
            <entry>
              <key><value>2</value></key>
              <ref bean="P002" />
            </entry>
            <entry>
              <key><value>3</value></key>
              <ref bean="P003" />
            </entry>
          </map>
        </property>
      </bean>
      <bean id="P001" class="domain.Profesor">
         <property name="id" value="1"/>
         <property name="cedula" value="101110111"/>
         <property name="nombre" value="Carlos Perez"/>
         <property name="titulo" value="Licenciado"/>
         <property name="area" value="Administracion"/>
         <property name="telefono" value="3456-7890"/>
      </bean>
      <bean id="P002" class="domain.Profesor">
         <property name="id" value="2"/>
         <property name="cedula" value="202220222"/>
         <property name="nombre" value="Luis Torres"/>
         <property name="titulo" value="Master"/>
         <property name="area" value="Economia"/>
         <property name="telefono" value="6677-3456"/>
      </bean>
      <bean id="P003" class="domain.Profesor">
         <property name="id" value="3"/>
         <property name="cedula" value="303330333"/>
         <property name="nombre" value="Juan Castro"/>
         <property name="titulo" value="Licenciado"/>
         <property name="area" value="Matematica"/>
         <property name="telefono" value="6755-7788"/>
      </bean>
    </beans>

Los aspectos importantes que se pueden observar en este archivo son la
declaración de una instancia (singleton) del repositorio de profesores y
la forma como se crean los objetos en forma dinámica.

Configuración del servidor
--------------------------

El servidor de servlets requiere del archivo de configuración de la
aplicación para conocer en donde se ubica la clase a ejecutar. Además
este archivo permite indicar la ubicación y nombre del archivo de
contexto. Estos archivos de configuración del servlet siempre se llaman
*web.xml* y deben residir en el directorio */tutorial4/root/WEB-INF/*.
Para este caso su contenido sería el siguiente:

::

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
       <url-pattern>/root/actualizarProfesor</url-pattern>
     </servlet-mapping>
     <servlet-mapping>
       <servlet-name>DetalleProfesor</servlet-name>
       <url-pattern>/root/detalleProfesor</url-pattern>
     </servlet-mapping>
     <servlet-mapping>
       <servlet-name>ListaProfesores</servlet-name>
       <url-pattern>/root/listaProfesores</url-pattern>
     </servlet-mapping>
    </web-app>

Ejecución del tutorial
======================

Este ejemplo se puede ejecutar bajo cualquier contenedor de Servlet. Por
ejemplo, para realizar la ejecución de pruebas se puede utilizar un
producto como el *Winstone Servlet Container* que permite ejecutar
servlets de forma muy sencilla. Se debe descargar el programa
*winstone-0.9.10.jar* y copiarlo en el directorio */tutorial4/*. Sin
embargo, para lograr que *Winstone* ejecute plantillas *JSP* es
necesario descargar algunas librerías adicionales que deben ser copiadas
en el directorio */tutorial4/lib*:

-  el-api-6.0.18.jar
-  jasper-6.0.18.jar
-  jasper-el-6.0.18.jar
-  jasper-jdt-6.0.18.jar
-  jsp-api-6.0.18.jar
-  jstl-api-1.2.jar
-  jstl-impl-1.2.jar
-  jtds-1.2.4.jar
-  juli-6.0.18.jar
-  servlet-api-2.5.jar
-  servlet-api.jar

Para ejecutar el servidor de servlets se puede crear un archivo de
instrucciones, llamado *run.bat*, similar al siguiente en el directorio
*/tutorial4/* (todo en una sola línea):

::

    java -jar winstone-0.9.10.jar --httpPort=8089 --commonLibFolder=lib
     --useJasper=true --webroot=root

Luego se puede acceder a la aplicación desde cualquier visualizador web
y apuntando a la dirección http://localhost:8089/listaProfesores
