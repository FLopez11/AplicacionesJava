Modelo/Vista/Controlador
========================

Spring provee muchas opciones para configurar una aplicación. La más
popular es usar archivos XML.

Capa del dominio
----------------

Se utilizará una única clase para ilustrar el desarrollo de una
aplicación MVC. En este caso será la clase de **Profesor.java**. Esta
clase se ubica en el directorio **src/domain** y su código es el
siguiente:

::

    package universidad.domain;
    import java.io.Serializable;
    public class Profesor implements Serializable {
     private String nombProf;
     private String idProf;
     private String tituloProf;
     public String getNombProf() {return nombProf;}
     public void setNombProf(String n) {nombProf=n;}
     public String getIdProf() {return idProf;}
     public void setIdProf(String id) {idProf=id;}
     public String getTituloProf() {return tituloProf;}
     public void setTituloProf(String tit)  {tituloProf = tit;}
    }

Capa de servicio
----------------

Aquí se utilizará una clase sencilla, llamada
**SimpleProfesorManager.java**, para configurar la capa de servicio.
Adicionalmente se utilizará una interfaz, llamada
**ProfesorManager.java**, en donde se definirán los métodos utilizados
por dicha clase. El código de dicha interfaz se localiza en el
directorio **src/service** y es el siguiente:

::

    package universidad.service;
    import java.io.Serializable;
    import java.util.List;

    import universidad.domain.Profesor;
    public interface ProfesorManager extends Serializable{
       public List<Profesor> getProfesores();
    }

El código de la clase **SimpleProfesorManager.java** se muestra a
continuación y se ubica en el mismo directorio:

::

    package universidad.service;
    import java.util.ArrayList;
    import java.util.List;
    import universidad.domain.Profesor;

    public class SimpleProfesorManager implements ProfesorManager {
      private List<Profesor> profesores;
      public List<Profesor> getProfesores() {
        return profesores;
      }
      public void setProfesores(List<Profesor> profesores) {
        this.profesores = profesores;
      }  
    }

El controlador
--------------

La capa de presentación consta de dos componentes, el controlador y la
vista. El controlador consta de una sola clase, llamada
**ProfesorController.java**, que carga el modelo de datos e invoca a la
vista, llamada **profesorView.jsp**. El código se muestra a continuación
y se ubica en el directorio **src/web**.:

::

    package universidad.web;
    import org.springframework.web.servlet.mvc.Controller;
    import org.springframework.web.servlet.ModelAndView;
    import javax.servlet.ServletException;
    import javax.servlet.http.HttpServletRequest;
    import javax.servlet.http.HttpServletResponse;
    import java.io.IOException;
    import java.util.Map;
    import java.util.HashMap;
    import org.apache.commons.logging.Log;
    import org.apache.commons.logging.LogFactory;
    import universidad.service.ProfesorManager;

    public class ProfesorController implements Controller {
     protected final Log logger = LogFactory.getLog(getClass());
     private ProfesorManager ProfesorManager;

     public ModelAndView handleRequest(HttpServletRequest request, 
       HttpServletResponse response)
       throws ServletException, IOException {

       String now = (new java.util.Date()).toString();
       logger.info("returning profesor view with " + now);

       Map<String, Object> myModel = new HashMap<String, Object>();
       myModel.put("now", now);
       myModel.put("profesores", this.ProfesorManager.getProfesores());

       return new ModelAndView("profesorView", "model", myModel);
     }

     public void setProfesorManager(ProfesorManager ProfesorManager) {
       this.ProfesorManager = ProfesorManager;
     }
    }

La vista
--------

El archivo que genera la vista se llama **profesorView.jsp** y consta de
varias etiquetas que permiten listar la información de profesores. El
código es el siguiente y este archivo se ubica en el directorio
**war/WEB-INF/jsp**.:

::

    <%@ include file="/WEB-INF/jsp/include.jsp" %>
    <html>
     <head><title><fmt:message key="title"/></title></head>
     <body>
       <h1><fmt:message key="heading"/></h1>
       <p><fmt:message key="mensaje"/> <c:out value="${model.now}"/></p>
       <h3>Profesores</h3>
       <table border="1">
         <tr><th>Nombre</th><th>Cedula</th><th>Titulo</th></tr>
         <c:forEach items="${model.profesores}" var="prof">
           <tr><td><c:out value="${prof.nombProf}"/></td>
           <td><c:out value="${prof.idProf}"/></td>
           <td><c:out value="${prof.tituloProf}"/></td></tr>
         </c:forEach>
       </table>
     </body>
    </html>

Como se puede observar es necesario contar con el archivo
**include.jsp** que se ubica en el directorio **war/WEB-INF/jsp** y cuyo
código es simplemente el siguiente:

::

    <%@ page session="false"%>
    <%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
    <%@ taglib prefix="fmt" uri="http://java.sun.com/jsp/jstl/fmt" %>

También existe un archivo que cuenta con algunos mensajes que se
despliegan en la página y se llama **messages.properties**. Este archivo
se ubica en el directorio **war/WEB-INF/classes** y su contenido es el
siguiente:

::

    title=Sistema Universitario
    heading=Sistema Universitario - Inicio
    mensaje=Bienvenido, la fecha actual es

Configuración
-------------

Para configurar la aplicación es necesario contar con un archivo
**web.xml** ubicado en el directorio **war/WEB-INF** con el siguiente
contenido.:

::

    <?xml version="1.0" encoding="UTF-8"?>
    <web-app version="2.4"
            xmlns="http://java.sun.com/xml/ns/j2ee"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee 
            http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd" >
     <servlet>
       <servlet-name>universidad</servlet-name>
       <servlet-class>org.springframework.web.servlet.DispatcherServlet
       </servlet-class>
       <load-on-startup>1</load-on-startup>
     </servlet>
     <servlet-mapping>
       <servlet-name>universidad</servlet-name>
       <url-pattern>*.htm</url-pattern>
     </servlet-mapping>
     <welcome-file-list>
       <welcome-file>
         index.jsp
       </welcome-file>
     </welcome-file-list>
    </web-app>

También se utilizará otro archivo de configuración para crear objetos de
ejemplo que permitan probar la aplicación. Esto se hará utilizando el
archivo **universidad-servlet.xml** que se ubica en el mismo directorio
**war/WEB-INF**. Su contenido es el siguiente:

::

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
     xsi:schemaLocation="http://www.springframework.org/schema/beans
     http://www.springframework.org/schema/beans/spring-beans-2.5.xsd">
     <bean id="profesorManager" 
       class="universidad.service.SimpleProfesorManager">
       <property name="profesores">
         <list>
           <ref bean="profesor1"/>
           <ref bean="profesor2"/>
           <ref bean="profesor3"/>
         </list>
       </property>
     </bean>
     <bean id="profesor1" class="universidad.domain.Profesor">
       <property name="nombProf" value="Juan Jimenez"/>
       <property name="idProf" value="303450678"/>
       <property name="tituloProf" value="Licenciado"/>
     </bean>
     <bean id="profesor2" class="universidad.domain.Profesor">
       <property name="nombProf" value="Pedro Perez"/>
       <property name="idProf" value="102340567"/>
       <property name="tituloProf" value="Maestria"/>
     </bean>
     <bean id="profesor3" class="universidad.domain.Profesor">
       <property name="nombProf" value="Luisa Linares"/>
       <property name="idProf" value="407860887"/>
       <property name="tituloProf" value="Licenciada"/>
     </bean>
     <bean id="messageSource" 
       class="org.springframework.context.support.ResourceBundleMessageSource">
       <property name="basename" value="messages"/>
     </bean>
     <bean name="/hello.htm" class="universidad.web.ProfesorController">
       <property name="profesorManager" ref="profesorManager"/>
     </bean>
     <bean id="viewResolver" 
       class="org.springframework.web.servlet.view.InternalResourceViewResolver">
       <property name="viewClass" 
                 value="org.springframework.web.servlet.view.JstlView"/>
       <property name="prefix" value="/WEB-INF/jsp/"/>
       <property name="suffix" value=".jsp"/>
     </bean>
    </beans>

Por último es necesario contar con un archivo inicial que reenvíe las
solicitudes que se realizan al sitio, este archivo se llama
**index.jsp** y se ubica en el directorio **war**. Su contenido es
simplemente:

::

    <%@ include file="/WEB-INF/jsp/include.jsp" %>
    <c:redirect url="/hello.htm"/>

Compilación y ejecución
-----------------------

Para compilar la aplicación es necesario copiar en el directorio
**war/WEB-INF/lib** las librerías **spring.jar** y **spring-webmvc.jar**
del framework Spring. También se requieren las librerías
**servlet-api-2.5.jar** y **commons-logging-1.1.1.jar** pero estas
pueden estar ubicados simplemente en el directorio **lib**.

Un script que permite compilar la aplicación tendría la siguiente forma
(todo en una sola línea):

::

    javac -classpath lib/servlet-api-2.5.jar;lib/commons-logging-1.1.1.jar;
      war/WEB-INF/lib/spring-webmvc.jar -d war/WEB-INF/classes 
      src/service/*.java src/domain/*.java src/web/*.java

Utilizando **winstone** se puede correr la aplicación utilizando
simplemente:

::

    java -jar winstone-0.9.10.jar --webroot=war --useJasper 

luego basta con apuntar el navegador a la dirección:

http://localhost:8080
