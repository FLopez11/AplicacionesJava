# Mapeo objeto/relacional

Este tutorial muestra la forma de desarrollar una aplicación que utilice
**modelo del dominio** como patrón de diseño de la capa de dominio, y
**mapeador objeto/relacional** para la capa de acceso a datos.

## Capa de lógica del dominio

Para implementar la capa de lógica del dominio se utilizará la técnica
de **modelo del dominio** tal como el tutorial anterior. En este caso el
modelo agrupa toda la lógica del dominio, pero no se encarga del acceso
a datos.

La primer clase necesaria consiste en la entidad profesor
**Profesor.java** y residirá en el directorio
**/tutorial5/src/domain/**. Esta clase es la que contendría la lógica
del dominio.

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

### El repositorio de datos

Para mantener almacenados y recuperar los diferentes objetos, se utiliza
el objeto llamado **ProfesorRepository.java** residente en el mismo
directorio **/tutorial5/src/domain/**. Sin embargo, este tipo de objeto
solamente es una interfase Java como se puede observar a continuación:

    package domain;
    import java.util.Map;
    import java.util.HashMap;
    import java.util.Collection;

    public interface ProfesorRepository {
      public boolean insertProfesor(Profesor prof);
      public boolean deleteProfesor(Profesor prof);
      public Profesor findProfesor(int id);
      public boolean updateProfesor(Profesor prof);
      public Collection findAllProfesor();
    }

### La fábrica de objetos

Generalmente cuando se elabora un modelo del dominio es importante crear
una clase aparte que se encargue de crear instancias de objetos. En este
caso se utilizará la clase **ProfesorFactory.java** y se ubicará en el
mismo directorio **/tutorial5/src/domain/**

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

### Compilando la capa del dominio

Se puede realizar la compilación de estas tres clases en forma separada
del resto del código. La siguiente instrucción para ejecutar la
compilación puede estar definida en un archivo
**compileDomainLayer.bat** residente en el directorio **/tutorial5/**
(todo en una sola línea):

    javac -d root/WEB-INF/classes src/domain/Profesor.java
      src/domain/ProfesorFactory.java src/domain/ProfesorRepository.java

Nota: La versión del JDK debe ser superior a 6.0

## Capa de presentación

El servicio de la universidad será implementado mediante
**controladores de página** (tal como se hizo en el tutorial anterior),
en donde cada página se implementa como un controlador individual. Igual
que antes, la clase general para definir los controladores se llama
**PageController.java** y debe residir en el directorio
**/tutorial5/src/display/**.

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
         WebApplicationContextUtils.getWebApplicationContext(
           getServletContext());
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

El primer controlador de página es el que permite mostrar el listado de
profesores. Este archivo se llama **ListaProfesores.java** y reside en
el mismo directorio **/tutorial5/src/display/**.

    package display;
    import java.util.*;
    import java.io.*;
    import javax.servlet.*;
    import javax.servlet.http.*;
    import org.springframework.web.context.*;
    import domain.ProfesorRepository;
    import domain.Profesor;
    import util.ProfesorDTO;
    import util.ProfesorAssembler;

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
                ProfesorDTO dto = ProfesorAssembler.CreateDTO(prof);
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

#### La plantilla JSP

Adicionalmente se utilizará, con en el tutorial anterior, una
**plantilla** JSP para realizar el formateo de página en código HTML. El
archivo **listaProfesores.jsp** se encarga de esta tarea y residirá en
el directorio **/tutorial5/root/**.

	<%@ page import="java.util.*" %>
	<%@ page import="util.*" %>
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
	    <tr><th>Nombre</th><th>T&iacute;tulo</th>
	        <th>Area</th><th>Acciones</th></tr>
	    </thead>
	    <tbody>
	    <% for(int i = 0, n = profs.size(); i < n; i++) {
	         ProfesorDTO prof = (ProfesorDTO) profs.get(i); %>
	        <tr><td><%= prof.getNombre() %></td>
	        <td><%= prof.getTitulo() %></td>
	        <td><%= prof.getArea() %></td>
	        <td><a href='/detalleProfesor?id=<%= prof.getId() %>'>
	              <input type="submit" value="Detalle"/></a>
	            <a href='/eliminarProfesor?id=<%= prof.getId() %>'>
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

#### Hoja de estilo

Con el fin de mejorar la apariencia de este ejemplo se ha desarrollado una pequeña hoja de estilo (css) que permite organizar mejor los datos de la tabla y el formulario. El archivo llamado *style.css* cuenta con el siguiente código y y reside en el directorio */tutorial5/root/*.:

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

El controlador de detalle de profesor presentará otra tabla HTML con la
información detallada del profesor. Este controlador es llamado
**DetalleProfesor.java** y se ubica en el mismo directorio
**/tutorial5/src/display/**. Es importante observar la utilización del
**id** del profesor para realizar la consulta al módulo de tabla.

    package display;
    import java.util.*;
    import java.io.*;
    import javax.servlet.*;
    import javax.servlet.http.*;
    import org.springframework.web.context.*;
    import domain.ProfesorRepository;
    import domain.Profesor;
    import util.ProfesorDTO;
    import util.ProfesorAssembler;

    public class DetalleProfesor extends PageController {

      public void doGet(HttpServletRequest request,
                        HttpServletResponse response)
        throws ServletException, IOException {
          ProfesorRepository profesores =
            (ProfesorRepository) context.getBean("profesorRepository");
        try {
          String id = request.getParameter("id");
          int idProf = Integer.parseInt(id); 
          Profesor prof = profesores.findProfesor(idProf);
          ProfesorDTO dto = ProfesorAssembler.CreateDTO(prof);
          request.setAttribute("profesor",dto);
          forward("/detalleProfesor.jsp",request,response);
        } catch (Exception e) {
          request.setAttribute("mensaje",e.getMessage());
          forward("/paginaError.jsp",request,response);
        }
      }
    }

#### Plantilla JSP

La plantilla JSP que genera el código HTML del detalle del profesor, se
presenta a continuación. El código de la plantilla se define en un
archivo llamado **detalleProfesor.jsp** que reside también en el
directorio **/tutorial5/root/**.

	<%@ page import="java.util.Map" %>
	<%@ page import="util.*" %>
	<html>
	  <head>
	    <meta http-equiv="Content-Type" content="text/html; 
	          charset=UTF-8"/>
	    <title>Sistema Universitario</title>
	    <link rel="stylesheet" href="style.css">
	  </head>
	  <h1>Sistema Universitario</h1>
	  <h2>Detalle de Profesor</h2>
	  <% ProfesorDTO prof = 
	    (ProfesorDTO)request.getAttribute("profesor"); %>
	  <form name="ActualizarProfesor" 
	        action="/actualizarProfesor" method="get">
	  <input type="hidden" name="id" value="<%= prof.getId() %>"/>
	  <table>
	    <thead>
	      <tr><th></th><th></th></tr>
	    </thead>
	    <tbody>
	    <tr><td>Nombre:</td><td><input type="text" name="nombre" 
	            value="<%= prof.getNombre() %>"/></td></tr>
	        <tr><td>C&eacute;dula:</td><td><input type="text" 
	            name="cedula" value="<%= prof.getCedula() %>"/>
	            </td></tr>
	        <tr><td>T&iacute;tulo:</td><td><input type="text" 
	            name="titulo" value="<%= prof.getTitulo() %>"/>
	            </td></tr>
	        <tr><td>Area:</td><td><input type="text" name="area" 
	            value="<%= prof.getArea() %>"/></td></tr>
	        <tr><td>Tel&eacute;fono:</td><td><input type="text" 
	            name="telefono" value="<%= prof.getTelefono() %>"/>
	            </td></tr>
	    </tbody>
	    <tfoot>
	    <tr><td><input type="submit" value="Actualizar" />
	        </td><td></td></tr>
	    </tfoot>
	  </table>
	  </form>
	</html>

### El controlador para actualizar información

Se presenta también el controlador de página que permite actualizar los
datos de un profesor. La lógica de este controlador se ubica en el
archivo **ActualizarProfesor.java** y reside en el directorio
**/tutorial5/src/display/**.

    package display;
    import java.util.*;
    import java.io.*;
    import javax.servlet.*;
    import javax.servlet.http.*;
    import org.springframework.web.context.*;

    import domain.ProfesorRepository;
    import domain.Profesor;

    import util.ProfesorDTO;
    import util.ProfesorAssembler;

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
            Profesor prof = profesores.findProfesor(idProf);
            try {
                if (cedula!=null) prof.setCedula(cedula);
                if (nombre!=null) prof.setNombre(nombre);
                if (titulo!=null) prof.setTitulo(titulo);
                if (area!=null) prof.setArea(area);
                if (telefono!=null) prof.setTelefono(telefono);
                    profesores.updateProfesor(prof);
            } catch (Exception e) {}
            response.sendRedirect("listaProfesores");
        } catch (Exception e) {
            request.setAttribute("mensaje",e.getMessage());
            forward("/paginaError.jsp",request,response);
        }
      }
    }

#### Plantilla JSP de mensaje de error

Se puede utilizar un archivo JSP adicional para desplegar los mensajes
de error. En particular este archivo será llamado **paginaError.jsp** y
residirá en el directorio **/tutorial5/root/**.

    <html>
      <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8"/>
        <title>Sistema Universitario</title>
      </head>
      <% String mensaje = (String)request.getAttribute("mensaje"); %>
      <h1>Error en operaci&oacute;n</h1>
      <p><%= mensaje %></p>
    </html>

### El DTO de profesor

En esta implementación también se utiliza una clase tipo DTO (Data
Transfer Object) que facilite el paso de información hacia las vistas de
datos. Para ello se utiliza la clase **ProfesorDTO.java** pero esta
clase residirá en el directorio **/tutorial5/src/util/**.

    package util;

    public class ProfesorDTO {
      private int id;
      private String cedula;
      private String nombre;
      private String titulo;
      private String area;
      private String telefono;

      public int getId() {return id;}
      public String getCedula() {return cedula;}
      public String getNombre() {return nombre;}
      public String getTitulo() {return titulo;}
      public String getArea() {return area;}
      public String getTelefono() {return telefono;}
      public void setId(int id) {this.id=id;}
      public void setCedula(String ced) {cedula=ced;}
      public void setNombre(String nom) {nombre=nom;}
      public void setTitulo(String tit) {titulo=tit;}
      public void setArea(String are) {area=are;}
      public void setTelefono(String tel) {telefono=tel;}
    }

#### El ensamblador del DTO

Adicionalmente es necesario contar con una clase que realice el
ensamblaje del DTO a partir de la entidad de profesor. Aquí se utiliza
la clase **ProfesorAssembler.java** residente en el mismo directorio
**/tutorial5/src/util/**.

    package util;
    import domain.Profesor;

    public class ProfesorAssembler {
      public static ProfesorDTO CreateDTO(Profesor prof) {
        ProfesorDTO dto = new ProfesorDTO();
        dto.setId(prof.getId());
        dto.setCedula(prof.getCedula());
        dto.setNombre(prof.getNombre());
        dto.setTitulo(prof.getTitulo());
        dto.setArea(prof.getArea());
        dto.setTelefono(prof.getTelefono());
        return dto;
      }
      public static void Update(Profesor prof, ProfesorDTO dto) {
        try {
          prof.setId(dto.getId());
          prof.setCedula(dto.getCedula());
          prof.setNombre(dto.getNombre());
          prof.setTitulo(dto.getTitulo());
          prof.setArea(dto.getArea());
          prof.setTelefono(dto.getTelefono());
        } catch (Exception e) {
        }
      }
    }

### Compilando la capa de presentación

Para compilar la capa de presentación es necesario contar con las
librerías del framework **Spring 3** (como se indicó antes) y con la
librería **servlet-api.jar** ubicadas en el directorio
**/tutorial5/root/WEB-INF/lib/**.

Específicamente las librerías necesarias son las siguientes:

-   servlet-api.jar
-   spring-asm-3.2.0.M1.jar
-   spring-beans-3.2.0.M1.jar
-   spring-context-3.2.0.M1.jar
-   spring-core-3.2.0.M1.jar
-   spring-expression-3.2.0.M1.jar
-   spring-jdbc-3.2.0.M1.jar
-   spring-orm-3.2.0.M1.jar
-   spring-tx.3.2.0.M1.jar
-   spring-web-3.2.0.M1.jar

La siguiente instrucción para ejecutar la compilación puede estar
definida en un archivo **compileDisplayLayer.bat** residente en el
directorio **/tutorial5/** (todo en una sola línea):

    javac -cp "root/WEB-INF/classes";"root/WEB-INF/lib/*"
      -d root/WEB-INF/classes src/display/PageController.java
      src/display/ActualizarProfesor.java src/display/DetalleProfesor.java
      src/display/ListaProfesores.java src/util/ProfesorAssembler.java
      src/util/ProfesorDTO.java

## Capa de acceso a datos

Para la capa de acceso a datos se utilizará el patrón de **mapeador de
datos**. Para ello se requiere las librerías **Spring** e **Hibernate**.
En forma conjunta estas dos librerías facilitan el desarrollo de este
tipo de aplicaciones.

Un DAO (Data Access Object) debe incluir todo el código para realizar la
conexión con la base de datos y ejecutar las instrucciones SQL de
consulta y/o actualización de datos. Generalmente escribir todo este
código desde el principio resulta muy tedioso. Es por eso que Spring
provee la clase **HibernateDaoSupport** que facilita en gran medida la
escritura de clases tipo DAO.

### El DAO de profesor

Para empezar se definirá la clase **ProfesorDAO.java** que residirá en
el directorio **/src/data**. Esta clase define el conjunto de
operaciones que se llevan a cabo sobre la base de datos y mediante
Hibernate. El contenido de dicho archivo sería el siguiente:

    package data;
    import java.util.Collection;
    import util.ProfesorDTO;
    import util.ProfesorAssembler;
    import org.springframework.orm.hibernate3.support.HibernateDaoSupport;

    public class ProfesorDAO extends HibernateDaoSupport {
     public boolean insert(ProfesorDTO profDTO) {
       getHibernateTemplate().saveOrUpdate(profDTO);
       return true;
     }
     public boolean delete(ProfesorDTO profDTO) {
       getHibernateTemplate().delete(profDTO);
       return true;
     }
     public ProfesorDTO findById(int id) {
       ProfesorDTO prof;
       prof = (ProfesorDTO)getHibernateTemplate().get(
               ProfesorDTO.class,new Integer(id));
       return prof;
     }
     public boolean update(ProfesorDTO profDTO) {
       getHibernateTemplate().saveOrUpdate(profDTO);
       return true;
     }
     public Collection findAll() {
       return getHibernateTemplate().find("from ProfesorDTO");
     }
    }

Como se puede observar aquí también se utiliza el DTO del profesor pero
en esta ocasión es para pasar los datos desde objetos del dominio a la
capa de datos.

#### Repositorio basado en DAO

Ahora es necesario asociar esta clase tipo DAO con el modelo del dominio
de la capa de presentación. Este trabajo lo lleva a cabo la clase
**ProfesorRepositoryDAOImpl.java**, que es una clase equivalente a la
utiliza en el modelo del dominio. Esta clase también reside en el
directorio **/src/data** y su contenido sería:

    package data;
    import java.util.Collection;
    import java.util.Iterator;
    import java.util.List;
    import java.util.ArrayList;
    import domain.ProfesorRepository;
    import util.ProfesorDTO;
    import util.ProfesorAssembler;
    import domain.Profesor;

    public class ProfesorRepositoryDAOImpl 
           implements ProfesorRepository {
     private ProfesorDAO profDAO;
     ProfesorRepositoryDAOImpl(ProfesorDAO profDAO) {
       this.profDAO = profDAO;
     }
     public boolean insertProfesor(Profesor prof) {
       ProfesorDTO profDTO = ProfesorAssembler.CreateDTO(prof);
       return (profDAO.insert(profDTO));
     }
     public boolean deleteProfesor(Profesor prof) {
       ProfesorDTO profDTO = ProfesorAssembler.CreateDTO(prof);
       return (profDAO.delete(profDTO));
     }
     public Profesor findProfesor(int id) {
       ProfesorDTO profDTO = profDAO.findById(id);
       if (profDTO!=null) {
         Profesor prof = new Profesor();
         System.out.println(profDTO.getNombre());
         ProfesorAssembler.Update(prof,profDTO);
         return prof;
       }
       return null;
     }
     public boolean updateProfesor(Profesor prof) {
       ProfesorDTO profDTO = ProfesorAssembler.CreateDTO(prof);
       return (profDAO.update(profDTO));
     }
     public Collection findAllProfesor() {
       Collection profsDTO = profDAO.findAll();
       List profList = new ArrayList();
       Iterator itr = profsDTO.iterator();
       while (itr.hasNext()) {
         Profesor prof = new Profesor();
         ProfesorDTO profDTO = (ProfesorDTO)itr.next();
         ProfesorAssembler.Update(prof,profDTO);
         profList.add(prof);
       }
       return profList;
     }
    }

### Base de datos

Se utilizará la misma base de datos SQLite del tutorial anterior para
administrar los datos. Dicha base de datos debe llevar por nombre
**universidad.sqlite** y debe estar ubicada en el directorio
**/tutorial5/root/database/**. El código SQL utilizado para
generar la tabla de profesores sería el siguiente:

    CREATE TABLE profesor (id INTEGER PRIMARY KEY, cedula VARCHAR, 
       nombre VARCHAR, titulo VARCHAR, area VARCHAR, telefono VARCHAR);
    INSERT INTO profesor VALUES(1,'101110111','Carlos Perez',
      'Licenciado','Administracion','3456-7890');
    INSERT INTO profesor VALUES(2,'202220222','Luis Torres',
                         'Master','Economia','6677-3456');
    INSERT INTO profesor VALUES(3,'303330333','Juan Castro',
      'Licenciado','Matematica','6755-7788');

Para administrar una base de datos SQLite se puede utilizar alguno de
los excelentes productos creados para ello, tal como **SQLiteman** ó el
plugin para Firefox llamado **SQLite Manager**

### Compilando la capa de datos

La compilación de la capa de datos también requiere las librerías de
**Spring 3**.

La siguiente instrucción para ejecutar la compilación puede estar
definida en un archivo **compileDataLayer.bat** residente en el
directorio **/tutorial5** (todo en una sola línea):

    javac -cp "root/WEB-INF/classes";"root/WEB-INF/lib/*" 
      -d root/WEB-INF/classes src/data/ProfesorRepositoryDAOImpl.java
       src/data/ProfesorDAO.java

Nota: La versión del JDK debe ser superior a 6.0

### Dialecto SQLite

Para que **Hibernate** reconozca cualquier base de datos es necesario
contar con un archivo de **dialecto** que le identifique a **Hibernate**
algunas características importantes del motor de bases de datos.
Extrañamente, la versión actual de **Hibernate** no cuenta con dicho
archivo de dialecto para SQLite. Sin embargo, resulta sencillo escribir
dicho archivo directamente.

A continuación se presenta el archivo **SQLDialect.java**, que reside en
el directorio **tutorial5/src/dialect**, y cuyo contenido es:

    package dialect;
    import java.sql.Types;
    import org.hibernate.dialect.Dialect;
    import org.hibernate.dialect.function.StandardSQLFunction;
    import org.hibernate.dialect.function.SQLFunctionTemplate;
    import org.hibernate.dialect.function.VarArgsSQLFunction;
    import org.hibernate.type.StandardBasicTypes;

    public class SQLiteDialect extends Dialect {
        public SQLiteDialect() {
            super();
            registerColumnType(Types.BIT, "integer");
            registerColumnType(Types.TINYINT, "tinyint");
            registerColumnType(Types.SMALLINT, "smallint");
            registerColumnType(Types.INTEGER, "integer");
            registerColumnType(Types.BIGINT, "bigint");
            registerColumnType(Types.FLOAT, "float");
            registerColumnType(Types.REAL, "real");
            registerColumnType(Types.DOUBLE, "double");
            registerColumnType(Types.NUMERIC, "numeric");
            registerColumnType(Types.DECIMAL, "decimal");
            registerColumnType(Types.CHAR, "char");
            registerColumnType(Types.VARCHAR, "varchar");
            registerColumnType(Types.LONGVARCHAR, "longvarchar");
            registerColumnType(Types.DATE, "date");
            registerColumnType(Types.TIME, "time");
            registerColumnType(Types.TIMESTAMP, "timestamp");
            registerColumnType(Types.BINARY, "blob");
            registerColumnType(Types.VARBINARY, "blob");
            registerColumnType(Types.LONGVARBINARY, "blob");
            // registerColumnType(Types.NULL, "null");
            registerColumnType(Types.BLOB, "blob");
            registerColumnType(Types.CLOB, "clob");
            registerColumnType(Types.BOOLEAN, "integer");

            registerFunction("concat", 
               new VarArgsSQLFunction(StandardBasicTypes.STRING, "",
                    "||", ""));
            registerFunction("mod", 
              new SQLFunctionTemplate(StandardBasicTypes.INTEGER,
                    "?1 % ?2"));
            registerFunction("substr", 
              new StandardSQLFunction("substr",
                    StandardBasicTypes.STRING));
            registerFunction("substring", 
              new StandardSQLFunction("substr",
                    StandardBasicTypes.STRING));
        }
        public boolean supportsIdentityColumns() {return true;}
        public boolean hasDataTypeInIdentityColumn() {return false;}
        public String getIdentityColumnString() {return "integer";}
        public String getIdentitySelectString() {
            return "select last_insert_rowid()";
        }
        public boolean supportsLimit() {return true;}
        public String getLimitString(String query, boolean hasOffset) {
            return new StringBuffer(query.length() + 20).append(query).append(
                    hasOffset ? " limit ? offset ?" : " limit ?").toString();
        }
        public boolean supportsTemporaryTables() {return true;}
        public String getCreateTemporaryTableString() {
            return "create temporary table if not exists";
        }
        public boolean dropTemporaryTableAfterUse() {return false;}
        public boolean supportsCurrentTimestampSelection() {return true;}
        public boolean isCurrentTimestampSelectStringCallable() {return false;}
        public String getCurrentTimestampSelectString() {
            return "select current_timestamp";
        }
        public boolean supportsUnionAll() {return true;}
        public boolean hasAlterTable() {return false;}
        public boolean dropConstraints() {return false;}
        public String getAddColumnString() {
            return "add column";
        }
        public String getForUpdateString() {return "";}
        public boolean supportsOuterJoinForUpdate() {return false;}
        public String getDropForeignKeyString() {
            throw new UnsupportedOperationException(
                    "No drop foreign key syntax supported by SQLiteDialect");
        }
        public String getAddForeignKeyConstraintString(String constraintName,
                String[] foreignKey, String referencedTable, String[] primaryKey,
                boolean referencesPrimaryKey) {
            throw new UnsupportedOperationException(
                    "No add foreign key syntax supported by SQLiteDialect");
        }
        public String getAddPrimaryKeyConstraintString(String constraintName) {
            throw new UnsupportedOperationException(
                    "No add primary key syntax supported by SQLiteDialect");
        }
        public boolean supportsIfExistsBeforeTableName() {return true;}
        public boolean supportsCascadeDelete() {return false;}
    }

### Compilando el dialecto

Para compilar el dialecto de **SQLite** se puede ejecutar el archivo
**compileDialect.bat** desde el directorio **tutorial5**, y con el
siguiente contenido:

    javac -cp "root/WEB-INF/classes";"root/WEB-INF/lib/*" 
    -d root/WEB-INF/classes src/dialect/SQLiteDialect.java

## Configuración de la aplicación

Este ejemplo se puede ejecutar bajo cualquier contenedor de Servlet. Por
ejemplo, para realizar la ejecución de pruebas se puede utilizar un
producto como el **Winstone Servlet Container** que permite ejecutar
servlets de forma muy sencilla. Se debe descargar el programa
**winstone-0.9.10.jar** y copiarlo en el directorio **/tutorial5/**. Sin
embargo (como antes), para lograr que **Winstone** ejecute plantillas
**JSP** es necesario descargar algunas librerías adicionales que deben
ser copiadas en el directorio **/tutorial5/lib**:

-   el-api-6.0.18.jar
-   jasper-6.0.18.jar
-   jasper-el-6.0.18.jar
-   jasper-jdt-6.0.18.jar
-   jsp-api-6.0.18.jar
-   jstl-api-1.2.jar
-   jstl-impl-1.2.jar
-   jtds-1.2.4.jar
-   juli-6.0.18.jar
-   servlet-api-2.5.jar
-   servlet-api.jar

Adicionalmente es necesario que en el directorio
**tutorial5/root/WEB-INF/lib** se encuentren las librerías que
conforman el paquete **Hibernate** y todas sus dependencias que
básicamente son:

-   antlr.jar
-   asm-attrs.jar
-   asm.jar
-   c3p0-0.9.0.jar
-   cglib-2.1.3.jar
-   commons-collections-3.1.jar
-   commons-dbcp-1.4.jar
-   commons-logging-1.1.1.jar
-   commons-pool-1.6.jar
-   dom4j-1.6.1.jar
-   hibernate-jpa-2.0-api-1.0.1.Final.jar
-   hibernate3.jar
-   javassist-3.12.0.GA.jar
-   jta-1.1.jar
-   slf4j-api-1.6.1.jar
-   sqlite-jdbc-3.6.0.jar

### Archivo de contexto

Es necesario crear un archivo de contexto en donde se realizará la
creación de una serie de objetos necesarios. Este archivo lleva por
nombre **context.xml** y residirá en el directorio
**tutorial5/root/WEB-INF** y su contenido sería:

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xmlns:context="http://www.springframework.org/schema/context"
      xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context-3.0.xsd">
      <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" 
            destroy-method="close">
            <property name="driverClassName" value="${jdbc.driverClassName}"/>
            <property name="url" value="${jdbc.url}"/>
            <property name="username" value="${jdbc.username}"/>
            <property name="password" value="${jdbc.password}"/>
      </bean>
      <bean id="sessionFactory" 
          class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">  
            <property name="dataSource" ref="dataSource" />  
            <property name="configurationClass" 
                value="org.hibernate.cfg.AnnotationConfiguration"/>
                <property name="configLocation">  
                <value>classpath:hibernate.cfg.xml</value>  
            </property>  
      </bean>
      <bean id="transactionManager" 
          class="org.springframework.orm.hibernate3.HibernateTransactionManager">  
          <property name="sessionFactory" ref="sessionFactory" />  
      </bean>
      <bean id="profesorDAO" class="data.ProfesorDAO">
        <property name="sessionFactory"><ref local="sessionFactory"/></property>
      </bean>
      <bean id="profesorRepository" class="data.ProfesorRepositoryDAOImpl">
        <constructor-arg>
          <ref bean="profesorDAO"/>
        </constructor-arg>
      </bean>
      <context:property-placeholder location="WEB-INF/jdbc.properties"/>
    </beans>

También es necesario el archivo **jdbc.properties** que reside en el
mismo directorio **tutorial5/root/WEB-INF** y cuyo contenido es:

    jdbc.driverClassName=org.sqlite.JDBC
    jdbc.url=jdbc:sqlite:root/database/universidad.sqlite
    jdbc.username=sa
    jdbc.password=root

### Archivo de configuración de Hibernate

**Hibernate 3** requiere de su propio archivo de configuración el cual
se llamará **hibernate.cfg.xml** y residirá en el directorio
**/tutorial5/root/WEB-INF/classes** y su contenido es:

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://hibernate.sourceforge.net/hibernate-configuration-3.0.dtd">
    <hibernate-configuration>
        <session-factory>
            <property name="show_sql">true</property>
            <property name="format_sql">true</property>
            <property name="dialect">dialect.SQLiteDialect</property>
            <mapping resource="ProfesorDTO.hbm.xml"/>
        </session-factory>
    </hibernate-configuration>

#### Metadatos para la clase Profesor

**Hibernate 3** utiliza archivos de metadatos para establecer la
relación entre los campos de cada tabla y los atributos de las clases.
En este caso se utilizará un archivo llamado **ProfesorDTO.hbm.xml** que
residirá en el mismo directorio
**tutorial5/root/WEB-INF/classes** y cuyo contenido sería:

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE hibernate-mapping PUBLIC "-//Hibernate/Hibernate Mapping DTD 3.0//EN" 
    "http://hibernate.sourceforge.net/hibernate-mapping-3.0.dtd">
    <hibernate-mapping>
    <class name="util.ProfesorDTO" table="profesor">
        <id name="id" column="id" type="int">
            <generator class="native"></generator>
        </id>
        <property name="cedula" column="cedula" type="string"></property>
        <property name="nombre" column="nombre" type="string"></property>
        <property name="titulo" column="titulo" type="string"></property>
        <property name="area" column="area" type="string"></property>
        <property name="telefono" column="telefono" type="string"></property>
    </class>
    </hibernate-mapping>

### El archivo de configuración de servlets

Por último, es necesario crear el archivo de definición de servlets para
esta aplicación. Como es costumbre su nombre será **web.xml**, su
ubicación será **tutorial5/root/WEB-INF**:

    <?xml version="1.0" encoding="ISO-8859-1"?>
    <web-app xmlns="http://java.sun.com/xml/ns/j2ee"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee
                           http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd"
       version="2.4">
     <display-name>Sistema Universitario</display-name>
     <description>Ejemplo de Mapeo Relacional/Objeto/description>
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

### Ejecución del Tutorial

Para ejecutar el servidor de servlets se puede crear un archivo de
instrucciones, llamado **run.bat** (todo en una sola línea), similar al siguiente en el
directorio **/tutorial5/**:

    java -jar winstone-0.9.10.jar --httpPort=8089 
         --commonLibFolder=lib --useJasper=true --webroot=root

Luego se puede acceder a la aplicación desde cualquier visualizador web
y apuntando a la dirección
[http://localhost:8089/listaProfesores](http://localhost:8089/listaProfesores)
