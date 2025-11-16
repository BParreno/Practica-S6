# Informe de Práctica: Orquestación con Docker Compose

## 1. Introducción y Objetivo

El objetivo de esta actividad fue construir una **arquitectura de aplicación web completa** utilizando **Docker Compose**. Esto implicó la orquestación de tres servicios (`wordpress`, `mysql`, y `phpmyadmin`) interconectados, la definición de una red personalizada y la garantía de la persistencia de datos y contenido mediante volúmenes.

---

## 2. Configuración de Arquitectura con Docker Compose

La infraestructura fue definida en un único archivo `docker-compose.yml`, donde se especifican los servicios, volúmenes y redes.

### 2.1. Archivo `docker-compose.yml`

```yaml
version: '3.8'

services:
  db:
    image: mysql:8.0
    container_name: wordpress-db
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: midbpass
      MYSQL_DATABASE: wordpress_db
      MYSQL_USER: wp_user
      MYSQL_PASSWORD: wppassword
    volumes:
      - db_data:/var/lib/mysql
    networks:
      - app-network

  wordpress:
    depends_on:
      db:
        condition: service_healthy 
    image: wordpress:latest
    container_name: wordpress-app
    restart: always
    ports:
      - "8000:80"
    environment:
      WORDPRESS_DB_HOST: db:3306 
      WORDPRESS_DB_NAME: wordpress_db
      WORDPRESS_DB_USER: wp_user
      WORDPRESS_DB_PASSWORD: wppassword
    networks:
      - app-network
    volumes:
      - ./wp-content:/var/www/html/wp-content

  phpmyadmin:
    depends_on:
      - db
    image: phpmyadmin/phpmyadmin
    container_name: wordpress-pma
    restart: always
    ports:
      - "8080:80"
    environment:
      PMA_HOST: db
      MYSQL_ROOT_PASSWORD: midbpass
    networks:
      - app-network

volumes:
  db_data:
  wp-content:

networks:
  app-network:
    driver: bridge
```
### 2.2. Elementos Clave y Propósito
| **Componente** | **Parámetro/Valor** | **Propósito** |
| :--- | :--- | :--- |
| **Servicio** | `db: mysql:8.0` | Servidor de base de datos. Se conecta a la red `app-network`. |
| **Servicio** | `wordpress: latest` | Aplicación web. Utiliza el nombre de servicio `db` para la conexión. |
| **Red** | `app-network` | Aísla los tres contenedores y permite que se comuniquen usando el nombre del servicio como *hostname*. |
| **Volumen** | `db_data` | Persistencia para los datos de MySQL, asegurando que no se pierdan al detener/eliminar el contenedor. |
| **Conexión** | `WORDPRESS_DB_HOST: db` | El nombre del servicio (`db`) permite la conexión interna sin exponer puertos innecesarios al host. |
| **Dependencia** | `condition: service_healthy` | Mecanismo de espera avanzado para que WordPress inicie **solo** cuando MySQL esté listo y saludable. |

---

## 3. Demostración

### 3.1. Ejecución y Verificación
El entorno se levantó con el comando único `docker-compose up -d`.
**1. Verificación de la Aplicación:** Se accedió a la interfaz de configuración de WordPress en `http://localhost:8000`, confirmando que el servicio `wordpress` se conectó correctamente a la base de datos `db`.
**2. Verificación de la Administración:** Se accedió a `phpMyAdmin` en `http://localhost:8080` y se inició sesión exitosamente, verificando la comunicación funcional entre los tres contenedores.

## 4. Conclusión
El uso de **Docker Compose** demostró ser fundamental para definir toda una infraestructura de múltiples servicios en un formato declarativo (`YML`). Esto automatiza la creación de las redes y volúmenes, asegurando una arquitectura robusta donde los datos son persistentes y los servicios están correctamente interconectados y aislados, elementos esenciales para el desarrollo profesional.
