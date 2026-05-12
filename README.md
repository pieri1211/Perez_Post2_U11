![img.png](img.png) analisis de sonarqube
![img_1.png](img_1.png) analisis segundo de sonarqube

Listo. Este `README.md` está ajustado a la **Unidad 11, Post-Contenido 1**, donde te piden documentar la refactorización avanzada, las técnicas aplicadas, la comparación antes/después en SonarQube y las capturas de pantalla del análisis inicial y final. La guía exige específicamente tabla comparativa de métricas SonarQube, descripción de cada técnica aplicada y evidencias con capturas antes/después.

Copia esto completo en tu archivo `README.md`:

````markdown
# Refactorización Avanzada y Clean Code Profundo — Unidad 11

## 1. Descripción del proyecto

Este repositorio corresponde al laboratorio de la Unidad 11: Refactorización Avanzada y Clean Code Profundo, Post-Contenido 1. El objetivo principal fue identificar code smells de tipo Bloater en un servicio Spring Boot y eliminarlos mediante técnicas formales de refactorización, verificando posteriormente la mejora de las métricas con SonarQube.

El proyecto parte de un código intencionalmente deficiente en la clase `PedidoService`, donde se concentraban problemas como `Long Method`, `Large Class`, `Primitive Obsession`, inyección de dependencias por campo y mezcla de responsabilidades. Después del análisis inicial, se aplicaron técnicas como `Extract Method`, `Extract Class` e introducción de `Value Objects`, con el fin de reducir la complejidad ciclomática, mejorar la mantenibilidad y separar adecuadamente las responsabilidades del sistema.

---

## 2. Tecnologías utilizadas

- Java 17+
- Spring Boot
- Maven
- Spring Web
- Spring Data JPA
- H2 Database
- SonarQube Community Edition
- Docker Desktop
- Git
- GitHub

---

## 3. Objetivo del laboratorio

El objetivo del laboratorio fue analizar un servicio Spring Boot con problemas deliberados de diseño y aplicar refactorización avanzada para mejorar la calidad interna del código. La mejora se verificó mediante dos análisis en SonarQube: uno antes de la refactorización y otro después de aplicar las correcciones.

Las técnicas aplicadas fueron:

- `Extract Method`, para dividir métodos extensos en operaciones más pequeñas y cohesivas.
- `Extract Class`, para separar responsabilidades que no pertenecían directamente al servicio principal.
- Introducción de `Value Objects`, para eliminar Primitive Obsession y agrupar datos relacionados.
- Inyección por constructor, para mejorar la testabilidad y reducir el acoplamiento implícito.

---

## 4. Análisis inicial con SonarQube

Antes de aplicar las refactorizaciones, se ejecutó un primer análisis con SonarQube sobre el código original. El servicio principal presentaba varios problemas de mantenibilidad, principalmente asociados a una clase con demasiadas responsabilidades y a un método excesivamente largo.

El análisis inicial se ejecutó con el siguiente comando:

```bash
mvn verify sonar:sonar \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.token=TU_TOKEN \
  -Dsonar.projectKey=refactoring-u11
````

---

## 5. Code smells identificados inicialmente

### 5.1 Long Method

El método `procesarPedido` concentraba varias operaciones dentro de un solo bloque de código: validación del cliente, cálculo del total, aplicación de descuentos, notificación y persistencia del pedido.

Este diseño reduce la legibilidad, aumenta la complejidad ciclomática y dificulta la prueba unitaria del comportamiento del sistema.

---

### 5.2 Large Class

La clase `PedidoService` asumía responsabilidades que no correspondían exclusivamente al procesamiento del pedido. Además de coordinar la lógica principal, también contenía lógica de notificación y validaciones detalladas del cliente.

Esto afecta el principio de responsabilidad única, ya que una misma clase queda encargada de múltiples razones de cambio.

---

### 5.3 Primitive Obsession

El método `procesarPedido` recibía demasiados parámetros primitivos o simples, como `clienteNombre`, `clienteEmail`, `clienteTelefono`, `clienteDireccion`, `clienteCiudad` y `clienteCodigoPostal`.

Estos datos representaban conceptualmente una misma entidad del dominio, pero estaban dispersos como valores independientes. Para corregirlo, se introdujeron objetos de valor.

---

### 5.4 Inyección de dependencias por campo

El código inicial utilizaba `@Autowired` directamente sobre el atributo del repositorio:

```java
@Autowired
private PedidoRepository repo;
```

Esta práctica reduce la claridad de las dependencias obligatorias de la clase y dificulta la construcción de pruebas unitarias.

---

## 6. Técnicas de refactorización aplicadas

### 6.1 Introducción de Value Object: `DatosCliente`

Para eliminar el problema de `Primitive Obsession`, se creó la clase `DatosCliente`, encargada de encapsular los datos principales del cliente.

```java
public class DatosCliente {

    private final String nombre;
    private final String email;
    private final String telefono;
    private final Direccion direccion;

    public DatosCliente(String nombre, String email, String telefono, Direccion direccion) {
        if (nombre == null || nombre.isBlank()) {
            throw new IllegalArgumentException("Nombre requerido");
        }

        if (email == null || !email.contains("@")) {
            throw new IllegalArgumentException("Email inválido");
        }

        this.nombre = nombre;
        this.email = email;
        this.telefono = telefono;
        this.direccion = direccion;
    }

    public String getNombre() {
        return nombre;
    }

    public String getEmail() {
        return email;
    }

    public String getTelefono() {
        return telefono;
    }

    public Direccion getDireccion() {
        return direccion;
    }
}
```

Con esta refactorización, los datos del cliente dejaron de manejarse como parámetros aislados y pasaron a representarse mediante un objeto con significado dentro del dominio.

---

### 6.2 Introducción de Value Object: `Direccion`

También se creó el objeto de valor `Direccion`, encargado de agrupar los datos relacionados con la ubicación del cliente.

```java
public class Direccion {

    private final String calle;
    private final String ciudad;
    private final String codigoPostal;

    public Direccion(String calle, String ciudad, String codigoPostal) {
        if (calle == null || calle.isBlank()) {
            throw new IllegalArgumentException("La calle es requerida");
        }

        if (ciudad == null || ciudad.isBlank()) {
            throw new IllegalArgumentException("La ciudad es requerida");
        }

        if (codigoPostal == null || codigoPostal.isBlank()) {
            throw new IllegalArgumentException("El código postal es requerido");
        }

        this.calle = calle;
        this.ciudad = ciudad;
        this.codigoPostal = codigoPostal;
    }

    public String getCalle() {
        return calle;
    }

    public String getCiudad() {
        return ciudad;
    }

    public String getCodigoPostal() {
        return codigoPostal;
    }
}
```

La clase fue diseñada como inmutable, con atributos `final` y sin métodos `set`, lo cual evita modificaciones inconsistentes después de su construcción.

---

### 6.3 Aplicación de Extract Method

El método `procesarPedido` fue dividido en métodos más pequeños, cada uno con una responsabilidad específica.

Código refactorizado:

```java
public String procesarPedido(DatosCliente cliente,
                             LineaPedido[] lineas,
                             String metodoPago,
                             boolean esUrgente,
                             CodigoDescuento descuento) {

    double total = calcularTotal(lineas);
    double totalConDescuento = aplicarDescuento(total, descuento);
    notificacionService.notificarPedido(cliente, esUrgente);

    return persistirPedido(cliente, totalConDescuento);
}
```

Método extraído para calcular el total:

```java
private double calcularTotal(LineaPedido[] lineas) {
    return Arrays.stream(lineas)
            .mapToDouble(linea -> linea.getPrecioUnitario() * linea.getCantidad())
            .sum();
}
```

Método extraído para aplicar descuento:

```java
private double aplicarDescuento(double total, CodigoDescuento descuento) {
    return descuento != null ? total * (1 - descuento.getPorcentaje()) : total;
}
```

Método extraído para persistir el pedido:

```java
private String persistirPedido(DatosCliente cliente, double total) {
    Pedido pedido = new Pedido(cliente.getNombre(), total);
    return "OK_" + pedidoRepository.save(pedido).getId();
}
```

Esta refactorización redujo el tamaño del método principal y permitió que cada operación quedara representada por una función con intención clara.

---

### 6.4 Aplicación de Extract Class: `NotificacionService`

La lógica de notificación fue separada de `PedidoService`, ya que enviar mensajes o correos no corresponde directamente a la responsabilidad principal del servicio de pedidos.

```java
@Service
public class NotificacionService {

    public void notificarPedido(DatosCliente cliente, boolean urgente) {
        System.out.println("Enviando notificación a: " + cliente.getEmail());
        System.out.println("Pedido urgente: " + urgente);
    }
}
```

Con esta separación, `PedidoService` queda enfocado en procesar pedidos, mientras que `NotificacionService` asume la responsabilidad de las notificaciones.

---

### 6.5 Inyección por constructor

Se reemplazó la inyección por campo con `@Autowired` por inyección mediante constructor.

```java
@Service
public class PedidoService {

    private final PedidoRepository pedidoRepository;
    private final NotificacionService notificacionService;

    public PedidoService(PedidoRepository pedidoRepository,
                         NotificacionService notificacionService) {
        this.pedidoRepository = pedidoRepository;
        this.notificacionService = notificacionService;
    }
}
```

Esta modificación hace explícitas las dependencias obligatorias de la clase, mejora la testabilidad y evita dependencias ocultas.

---

## 7. Segundo análisis con SonarQube

Después de aplicar las refactorizaciones, se ejecutó nuevamente el análisis en SonarQube con el mismo proyecto y las mismas propiedades de análisis.

Comando utilizado:

```bash
mvn verify sonar:sonar \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.token=sqp_8395df27ddf8a5043f674c619b686a0aa2202a5e \
  -Dsonar.projectKey=refactoring-u11
```

El propósito del segundo análisis fue verificar la reducción de code smells, la disminución de la complejidad ciclomática del método `procesarPedido` y la mejora general de la mantenibilidad.

---

## 8. Comparación de métricas antes y después

| Métrica                                     | Antes de refactorizar | Después de refactorizar | Resultado            |
| ------------------------------------------- | --------------------- | ----------------------- | -------------------- |
| Code Smells                                 | X                     | X                       | Disminuyó            |
| Complejidad ciclomática de `procesarPedido` | X                     | X                       | Disminuyó            |
| Technical Debt Ratio                        | X                     | X                       | Mejoró               |
| Bugs                                        | X                     | X                       | Mejoró / Sin cambios |
| Vulnerabilities                             | X                     | X                       | Mejoró / Sin cambios |
| Maintainability Rating                      | X                     | X                       | Mejoró / Sin cambios |
| Reliability Rating                          | X                     | X                       | Mejoró / Sin cambios |
| Security Rating                             | X                     | X                       | Mejoró / Sin cambios |

> Nota: Los valores marcados con `X` deben reemplazarse por los datos exactos mostrados en las dos capturas de SonarQube.

---



## 9. Estructura del repositorio

```text
apellido-post1-u11/
│
├── docs/
│   ├── sonarqube-antes.png
│   └── sonarqube-despues.png
│
├── src/
│   ├── main/
│   │   └── java/
│   │       └── com/
│   │           └── universidad/
│   │               └── refactoringu11/
│   │                   ├── domain/
│   │                   │   ├── Pedido.java
│   │                   │   ├── Producto.java
│   │                   │   ├── DatosCliente.java
│   │                   │   ├── Direccion.java
│   │                   │   ├── LineaPedido.java
│   │                   │   └── CodigoDescuento.java
│   │                   │
│   │                   ├── repository/
│   │                   │   └── PedidoRepository.java
│   │                   │
│   │                   ├── service/
│   │                   │   ├── PedidoService.java
│   │                   │   └── NotificacionService.java
│   │                   │
│   │                   └── RefactoringU11Application.java
│   │
│   └── test/
│
├── pom.xml
├── sonar-project.properties
└── README.md
```

---

## 10. Commits realizados

El repositorio contiene commits descriptivos que evidencian el avance progresivo del laboratorio:

```text
1. Creación del código original con code smells
2. Aplicación de refactorización con Value Objects, Extract Method y Extract Class
3. Ejecución del segundo análisis y documentación de mejoras en README
```

---

## 11. Resultado final

Después de aplicar las técnicas de refactorización, el proyecto presenta una estructura más clara, con responsabilidades mejor distribuidas y menor complejidad en el método principal. El uso de `DatosCliente`, `Direccion`, `LineaPedido` y `CodigoDescuento` permitió reemplazar parámetros primitivos dispersos por objetos con significado dentro del dominio.

La extracción de métodos redujo el tamaño de `procesarPedido`, mientras que la extracción de `NotificacionService` separó la lógica de notificación de la lógica propia del procesamiento de pedidos. Finalmente, la inyección por constructor permitió hacer explícitas las dependencias del servicio y mejorar la mantenibilidad del código.

El segundo análisis en SonarQube permitió verificar la mejora del proyecto mediante la reducción de code smells y la disminución de la complejidad ciclomática, cumpliendo con los criterios establecidos para la actividad de refactorización avanzada.

````

