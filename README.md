# Resultados del reconocimiento: Clínica Santa Lucía

Reconocimiento del canal digital simulado para el examen práctico de la Unidad II (SI-084). **Este documento presenta resultados y datos de entrada para elaborar el plan de auditoría; no constituye el plan.** `PLAN.md` conserva la plantilla original sin completar.

## Contexto y límites

- Solicitud: determinar mediante una auditoría posterior si el canal digital protege los datos de salud de los pacientes. Periodo a revisar: **01/01/2026 al 30/09/2026**.
- Incidente comunicado: el **08/09/2026**, una paciente vio el hemograma de otra. WebSur lo atribuyó a caché, sin informe de sustento. **La causa no se ha verificado**.
- Poblaciones declaradas: **3 850 citas al mes**, **12 400 pacientes con cuenta** y **214 postulaciones con CV en 2026**. Son cifras de la solicitud, no mediciones del reconocimiento.
- Autorización: solicitudes GET y HEAD a páginas públicas de `127.0.0.1`, puertos 8080–8090, conforme a `documentos/ALCANCE-AUTORIZADO.md`. No se enviaron formularios, no se probaron credenciales, no se descargaron respaldos y no se consultó `/v1/`.
- El paquete extraído tenía permisos `700`, que hicieron que nginx devolviera `403` al principio. Para obtener la simulación prevista, se levantó **una copia temporal** de `docker-compose.yml` e `infraestructura/` con permisos de lectura para nginx. No se cambió el contenido de los archivos originales. Las evidencias E1–E8 se generaron contra esa copia y sustituyeron las salidas `403` iniciales.

## Resultados E1–E8

| Prueba | Observación verificable | Declaración del proveedor |
|---|---|
| **E1 · WSTG-INFO-04** | Respondieron **3 servicios**: portal `:8080`, pruebas `:8081` y API `:8088` (HTTP 200). Pruebas anuncia una **copia de producción restaurada el 12/09/2026** y un usuario de pruebas. La raíz de la API indica `api-citas` **1.4.0**, entorno `produccion` y enumera tres endpoints `/v1/`; **no se consultaron**. | **D1 contradicha**: declara un solo activo y ningún otro entorno o servicio. |
| **E2 · WSTG-INFO-02** | La respuesta HEAD del portal anuncia `Server: nginx/1.20.2` y `X-Powered-By: PHP/7.4.3`. PHP 7.4 terminó su soporte el **28/11/2022**. nginx 1.20 dejó de ser la rama estable vigente al publicarse 1.22.0 el **24/05/2022**; esta fecha indica sustitución de la rama estable, no una fecha formal de fin de soporte publicada por nginx. | **D2 contradicha por las versiones anunciadas**. La cabecera PHP es una declaración del servidor simulado, no una comprobación del binario PHP instalado. |
| **E3 · WSTG-INFO-03** | `robots.txt` contiene **4** rutas `Disallow`: `/admin/`, `/respaldos/`, `/resultados/` y `/wp-content/plugins/wp-file-manager/`. `/.well-known/security.txt` responde **404**. `robots.txt` revela rutas de interés; no impone control de acceso. | No hay declaración D específica. Refuerza el contraste con **D5** y **D6**; esas rutas se observaron directamente en E7. |
| **E4 · WSTG-INFO-05** | El HTML incluye **4 comentarios con información interna**: contacto de WebSur; usuario `admin_pruebas`; dirección interna del LIS `10.20.0.15:8090`; y patrón `ver.php?orden=NNNNN` para resultados. Además, el formulario de resultados informa que la contraseña inicial es la fecha de nacimiento en formato `ddmmaaaa`. La etiqueta `generator` anuncia WordPress **5.8.1**. | **D8 contradicha**: el código publicado sí contiene comentarios y datos internos. La forma de la contraseña y el patrón de URL generan riesgos que deberán evaluarse después; aquí no se probaron. |
| **E5 · WSTG-INFO-08** | Las URL de recursos anuncian tema `medicalpro` **2.1.0**, `contact-form-7` **5.3.1**, jQuery **3.4.1** y un recurso de `wp-file-manager` con `ver=6.8`. E4 anuncia WordPress **5.8.1**, frente a **6.6** declarada. La referencia `ver=6.8` de un recurso **no confirma por sí sola** la versión instalada del plugin. El plugin File Manager anterior a 6.9 está asociado a **CVE-2020-25213**; su aplicabilidad a este entorno queda por validar en una fase autorizada. | **D3 contradicha** al menos para WordPress: 5.8.1 no coincide con la versión 6.6 declarada. La actualización mensual de cada componente no se pudo comprobar. |
| **E6 · WSTG-INFO-06** | Hay **4 formularios**: citas, resultados, postulaciones y administración. **3** piden datos personales (citas, resultados y postulaciones); citas solicita síntomas y especialidad (**datos de salud**), postulaciones admite archivo CV y citas usa **GET** con nombres, DNI, fecha de nacimiento, teléfono, correo y motivo de consulta. Los **4** `action` usan `http://`. La casilla `acepto` de citas aparece sin texto asociado en la salida observada. | **D7 contradicha como afirmación de diseño**: el formulario de citas enviaría datos personales y de salud por GET/HTTP. No se envió ningún formulario ni se constató una fuga real. |
| **E7 · WSTG-CONF-04/05** | `/respaldos/` muestra un índice con **4 nombres de archivos**: un CSV de citas, dos `.sql.gz` y un `.php.bak`; **no se descargó ninguno**. `/admin/` responde **HTTP 200** desde el acceso local que representa el lado público de la simulación. | **D5 y D6 contradichas en la simulación**: el panel responde y los respaldos figuran dentro del árbol web. No se comprobó acceso desde una red real externa a WebSur. |
| **E8 · WSTG-CONF-07/12 y WSTG-SESS-02** | Faltan las **4 cabeceras** revisadas: HSTS, CSP, X-Frame-Options y X-Content-Type-Options. `PHPSESSID` carece de **Secure, HttpOnly y SameSite**. El intento HTTPS a `127.0.0.1:443` devuelve **HTTP 000**. | **D4 contradicha para la simulación local**: el portal responde por HTTP sin esas cabeceras ni atributos. El nombre de dominio declarado no se probó porque queda fuera del alcance autorizado. |

## Fuentes para las comprobaciones externas

- [PHP: ramas sin soporte](https://www.php.net/eol.php): PHP 7.4 terminó soporte el 28/11/2022.
- [nginx: noticias de 2021](https://nginx.org/2021.html) y [noticias de 2022](https://nginx.org/2022.html): 1.20.2 se publicó el 16/11/2021 y 1.22.0 estable el 24/05/2022.
- [NVD: CVE-2020-25213](https://nvd.nist.gov/vuln/detail/CVE-2020-25213): afecta a `wp-file-manager` anterior a 6.9. La coincidencia de un parámetro de URL no demuestra que el código vulnerable esté instalado o sea explotable.

## Insumos para el futuro plan

Las observaciones permiten priorizar, sin afirmar vulnerabilidades todavía verificadas: (1) segregación y exposición del entorno de pruebas y de la API, E1; (2) acceso a resultados de otra persona, contraseña inicial predecible y patrón de dirección, E4 y E6, a la luz del incidente de setiembre; (3) tratamiento de datos de salud por GET/HTTP, E6 y E8; (4) exposición de respaldos y administración, E3 y E7; y (5) mantenimiento de componentes, E2 y E5. El plan debe incluir los **tres servicios encontrados**, las poblaciones de la solicitud y las pruebas posteriores sujetas a autorización nueva.

Cada archivo `evidencias/E*.txt` contiene la salida literal de la prueba indicada. `evidencias/SHA256SUMS.txt` sella esas ocho salidas; se excluye a sí mismo para evitar una referencia circular.
