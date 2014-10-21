# Controlador de aplicación con Spring

Spring provee muchas opciones para configurar una aplicación. La más popular es usar archivos XML.

## Capa del dominio

Se utilizará una única clase para ilustrar el desarrollo de una aplicación MVC. En este caso será la clase de **Profesor.java**. Esta clase se ubica en el directorio **src/domain** y su código es el siguiente:

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

## Capa de servicio

Aquí se utilizará una clase sencilla, llamada **SimpleProfesorManager.java**, para configurar la capa de servicio. Adicionalmente se utilizará una interfaz, llamada **ProfesorManager.java**, en donde se definirán los métodos utilizados por dicha clase. El código de dicha interfaz se localiza en el directorio **src/service** y es el siguiente:

	package service;
	import java.io.Serializable;
	import java.util.List;
	import domain.Profesor;
	public interface ProfesorManager extends Serializable{
	   public List<Profesor> getProfesores();
	   public Profesor getById(String id);
	}

El código de la clase **SimpleProfesorManager.java** se muestra a continuación y se ubica en el mismo directorio:

	package service;
	import java.util.ArrayList;
	import java.util.List;
	import java.util.Iterator;
	import domain.Profesor;
	public class SimpleProfesorManager implements ProfesorManager {
	  private List<Profesor> profesores;
	  public List<Profesor> getProfesores() {
	    return profesores;
	  }
	  public void setProfesores(List<Profesor> profesores) {
	    this.profesores = profesores;
	  }
	  public Profesor getById(String id) {
	    Iterator itr = profesores.iterator();
	    Profesor prof;
	    while (itr.hasNext()) {
	      prof = (Profesor)itr.next();
	      if (prof.getIdProf().equals(id)) return prof;
	    }
	    return null;
	  }
	}

## El controlador

La capa de presentación consta de dos componentes, el controlador y la vista. El controlador consta de una sola clase, llamada **ProfesorController.java**, que carga el modelo de datos e invoca a las vistas, llamada **listaProfesores.jsp** y **detalleProfesor.jsp**. El código se muestra a continuación y se ubica en el directorio **src/display**.:

	package display;

	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.stereotype.Controller;
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RequestMethod;
	import org.springframework.web.bind.annotation.PathVariable;
	import org.springframework.web.bind.annotation.RequestParam;
	import org.springframework.web.servlet.ModelAndView;

	import java.util.Map;
	import java.util.HashMap;

	import service.ProfesorManager;
	import domain.Profesor;

	@Controller
	@RequestMapping("/profesor")
	public class ProfesorController {
	 
	 @Autowired
	 private ProfesorManager ProfesorManager;
	 
	 @RequestMapping(value="/listado", method = RequestMethod.GET)
	 public ModelAndView listado() {
	   Map<String, Object> myModel = new HashMap<String, Object>();
	   myModel.put("profesores", this.ProfesorManager.getProfesores());
	   return new ModelAndView("listaProfesores", "model", myModel);
	 }
	 
	 @RequestMapping(value="/detalleProfesor/{idProf}", method = RequestMethod.GET)
	 public ModelAndView detalleProfesor(@PathVariable("idProf") String id) {
	   Profesor prof = this.ProfesorManager.getById(id);
	   if (prof == null)
	     System.out.println("NULO");
	   return new ModelAndView("detalleProfesor", "profesor", prof);
	 }
	 
	 @RequestMapping(value="/actualizarProfesor/{idProf}", params = { "id", "nombre", "titulo", "cedula" }, method = RequestMethod.GET)
	 public String actualizarProfesor(
	     @PathVariable("idProf") String id, 
	     @RequestParam("nombre") String nombre,
	     @RequestParam("titulo") String titulo,
	     @RequestParam("cedula") String cedula
	     
	  ) {
	   Profesor prof = this.ProfesorManager.getById(id);
	   prof.setNombProf(nombre);
	   prof.setTituloProf(titulo);
	   prof.setIdProf(cedula);
	   return "forward:/profesor/listado";
	 }

	 public void setProfesorManager(ProfesorManager ProfesorManager) {
	   this.ProfesorManager = ProfesorManager;
	} }

## La vistas

El archivo que genera la vista de profesores se llama **listaProfesores.jsp** y consta de varias etiquetas que permiten listar la información de profesores. El código es el siguiente y este archivo se ubica en el directorio **/root/pages**.:

	<%@ include file="/pages/include.jsp" %>
	<html>
	  <head>
	    <title>Sistema Universitario</title>
	    <link rel="stylesheet" href="/resources/style.css">
	    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
	  </head>
	  <h1>Sistema Universitario</h1>
	  <h2>Listado de profesores</h2>
	   <table>
	     <thead>
	     <tr><th>Nombre</th><th>Cedula</th><th>Titulo</th><th>Acciones</th></tr>
	     </thead>
	     <tbody>
	     <c:forEach items="${model.profesores}" var="prof">
	       <tr><td>${prof.nombProf}</td>
	       <td>${prof.idProf}</td>
	       <td>${prof.tituloProf}</td>
	       <td><a href='/profesor/detalleProfesor/${prof.idProf}'>
	              <input type="submit" value="Detalle"/></a>
	            <a href='/profesor/eliminarProfesor/${prof.idProf}'>
	              <input type="submit" value="Eliminar"/></a></td></tr>
	     </c:forEach>
	     </tbody>
	     <tfoot>
	      <tr><td><a href='/profesor/agregarProfesor'>
	        <input type="submit" name="action" value="Agregar"/></a>
	      </td><td></td><td></td><td></td></tr>
	    </tfoot>
	  </table>
	 </body>
	</html>

Como se puede observar es necesario contar con el archivo **include.jsp** que se ubica en el mismo directorio **root/pages** y cuyo código es simplemente el siguiente:

	<%@ page session="false"%>
	<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
	<%@ taglib prefix="fmt" uri="http://java.sun.com/jsp/jstl/fmt" %>

El otro archivo de vista es el que genera el detalle del profesor, es llamado **detalleProfesor.jsp** y se encuentra ubicado en el mismo directorio **/root/pages**

	<%@ include file="/pages/include.jsp" %>
	<html>
	  <head>
	    <title>Sistema Universitario</title>
	    <link rel="stylesheet" href="/resources/style.css">
	    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
	  </head>
	  <h1>Sistema Universitario</h1>
	  <h2>Detalle de Profesor</h2>
	  <form name="ActualizarProfesor" action="/profesor/actualizarProfesor/${profesor.idProf}" method="get">
	  <table style="width:400px;">
	    <thead>
	    <tr><th></th><th></th></tr>
	    </thead>
	    <tbody>
	    <input type="hidden" name="id" value="${profesor.idProf}"/>
	    <tr><td>Nombre:</td><td>
	      <input type="text" name="nombre" value="${profesor.nombProf}"/></td></tr>
	    <tr><td>Cedula:</td><td>
	      <input type="text" name="cedula" value="${profesor.idProf}"/></td></tr>
	    <tr><td>Titulo:</td><td>
	      <input type="text" name="titulo" value="${profesor.tituloProf}"/></td></tr>
	   </td></tr>
	    </tbody>
	    <tfoot>
	    <tr><td><input type="submit" value="Actualizar" /></td><td></td></tr>
	    </tfoot>
	   </tbody>
	  </table>
	  </form>
	</html>

## Configuración

Para configurar la aplicación es necesario contar con un archivo **web.xml** ubicado en el directorio **root/WEB-INF** con el siguiente contenido.:

	<web-app id="WebApp_ID" version="2.4"
	    xmlns="http://java.sun.com/xml/ns/j2ee" 
	    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	    xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee 
	    http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd">
	 
	    <display-name>Sistema Universitario</display-name>

	   <servlet>
	      <servlet-name>universidad</servlet-name>
	      <servlet-class>
	         org.springframework.web.servlet.DispatcherServlet
	      </servlet-class>
	      <load-on-startup>1</load-on-startup>
	   </servlet>
	 
	   <servlet-mapping>
	      <servlet-name>universidad</servlet-name>
	      <url-pattern>/</url-pattern>
	   </servlet-mapping>

	  <context-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>/WEB-INF/universidad-servlet.xml</param-value>
		</context-param>

		<listener>
			<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
		</listener>
	</web-app>

También se utilizará otro archivo de configuración para crear objetos de ejemplo que permitan probar la aplicación. Esto se hará utilizando el archivo **universidad-servlet.xml** que se ubica en el mismo directorio **root/WEB-INF**. Su contenido es el siguiente:

	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"  
	   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	   xmlns:context="http://www.springframework.org/schema/context" 
	   xmlns:mvc="http://www.springframework.org/schema/mvc"
	   xsi:schemaLocation="http://www.springframework.org/schema/beans  
	   http://www.springframework.org/schema/beans/spring-beans.xsd 
	   http://www.springframework.org/schema/context 
	   http://www.springframework.org/schema/context/spring-context.xsd
	   http://www.springframework.org/schema/mvc 
	   http://www.springframework.org/schema/mvc/spring-mvc-3.0.xsd">

		<context:component-scan base-package="display" />

		<bean id="profesorManager"
		class="service.SimpleProfesorManager">
		<property name="profesores">
			<list>
				<ref bean="profesor1"/>
				<ref bean="profesor2"/>
				<ref bean="profesor3"/>
			</list>
		</property>
		</bean>
		<bean id="profesor1" class="domain.Profesor">
			<property name="nombProf" value="Juan Jimenez"/>
			<property name="idProf" value="303450678"/>
			<property name="tituloProf" value="Licenciado"/>
		</bean>
		<bean id="profesor2" class="domain.Profesor">
			<property name="nombProf" value="Pedro Perez"/>
			<property name="idProf" value="102340567"/>
			<property name="tituloProf" value="Maestria"/>
		</bean>
		<bean id="profesor3" class="domain.Profesor">
			<property name="nombProf" value="Luisa Linares"/>
			<property name="idProf" value="407860887"/>
			<property name="tituloProf" value="Licenciada"/>
		</bean>
		<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		  <property name="prefix" value="/pages/"/>
		  <property name="suffix" value=".jsp"/> 
		</bean>
		
		<mvc:resources mapping="/resources/**" location="/web-resources/"/>
		<mvc:annotation-driven />
	</beans>

Por último es necesario contar con una hoja de estilo que se ubicaría en el directorio **/root/web-resources** y se llamaría **style.css**:

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

## Ejecución

Utilizando **winstone** se puede correr la aplicación utilizando simplemente:

    java -jar winstone-0.9.10.jar --httpPort=8089 --commonLibFolder=lib --useJasper=true --webroot=root 

luego basta con apuntar el navegador a la dirección:

[http://localhost:8089/profesor/listado](http://localhost:8089/profesor/listado)
