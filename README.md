# Laboratorio 8: Infrastructure as Code con AWS CloudFormation y Autenticación Serverless

En este laboratorio exploraremos el concepto de Infrastructure as Code (IaC) utilizando AWS CloudFormation para desplegar una infraestructura serverless completa. Implementaremos la misma funcionalidad del Laboratorio 7 pero utilizando un template de CloudFormation, y extenderemos la aplicación agregando autenticación JWT con un sistema de login y un Lambda Authorizer para proteger nuestras APIs.

El sistema de usuarios implementado es básico y tiene como único propósito demostrar cómo restringir el acceso a ciertos endpoints de la API. Los usuarios no representan pilotos ni otros roles específicos, simplemente son credenciales para controlar el acceso a la funcionalidad de creación de naves espaciales.

> ⚠️ **Importante**: Este laboratorio provee una sección de troubleshooting para enfrentar los problemas que está considerado vayan a surgir durante el desarrollo. Las fallas esperadas son parte del aprendizaje y se espera que los estudiantes las resuelvan.

## Marco Teórico

### ¿Qué es Infrastructure as Code (IaC)?

Infrastructure as Code (IaC) es la práctica de gestionar y aprovisionar recursos de infraestructura mediante archivos de definición legibles por máquina, en lugar de configuración manual o herramientas interactivas. IaC permite versionar, reproducir y automatizar el despliegue de infraestructura de manera consistente.

### AWS CloudFormation

AWS CloudFormation es un servicio que te ayuda a modelar y configurar tus recursos de AWS para que puedas dedicar menos tiempo a gestionar esos recursos y más tiempo a centrarte en las aplicaciones que se ejecutan en AWS.

**Características principales:**
- **Modelado**: Define todos los recursos de AWS en formato JSON o YAML
- **Provisión automatizada**: CloudFormation se encarga del orden y dependencias
- **Control de versiones**: Gestiona cambios en la infraestructura como código
- **Rollback automático**: Revierte cambios si algo falla
- **Reproducibilidad**: Despliega la misma infraestructura múltiples veces

**Conceptos clave:**
- **Template**: Archivo JSON o YAML que describe los recursos
- **Stack**: Conjunto de recursos creados a partir de un template
- **Parameters**: Variables que permiten personalizar el template
- **Outputs**: Valores que el template devuelve después de la creación
- **Resources**: Recursos de AWS que se crearán

### Autenticación JWT en Arquitecturas Serverless

JSON Web Token (JWT) es un estándar abierto (RFC 7519) que define una forma compacta y autocontenida de transmitir información de forma segura entre partes como un objeto JSON.

**Características principales:**
- **Autocontenido**: Contiene toda la información necesaria sobre el usuario
- **Stateless**: No requiere almacenamiento del lado del servidor
- **Seguro**: Firmado digitalmente para verificar integridad

**Estructura de un JWT:**
```
Header.Payload.Signature
```

### Lambda Authorizers

Los Lambda Authorizers son funciones Lambda que controlan el acceso a las APIs de API Gateway. Permiten implementar lógica de autorización personalizada usando tokens de portador como JWT sin necesidad de agregar lógica de autorización directamente en las funciones Lambda que manejan las solicitudes.

**Tipos de Lambda Authorizers:**
- **Token-based**: Autoriza basándose en un token (como JWT)
- **Request-based**: Autoriza basándose en parámetros de la solicitud

## Pasos iniciales

### Requisitos previos

Antes de comenzar, asegúrate de tener:
1. Cuenta de AWS
2. Conocimientos básicos de YAML
3. Conocimientos básicos de JWT
4. Completado el Laboratorio 7
5. Navegador web y acceso a Postman

## 1. Despliegue de la infraestructura con CloudFormation

### Paso 1.1: Crear el stack de CloudFormation

1. **Accede a CloudFormation**:
   - En la **consola de AWS**, busca y selecciona el servicio **CloudFormation**
   - Verifica que estés en la región **sa-east-1** (Sudamérica - São Paulo)
   
2. **Inicia la creación del stack**:
   - Haz clic en **"Create stack"** → **"With new resources (standard)"**
   
3. **Sube el template**:
   - En la sección **"Specify template"**:
     - Selecciona **"Upload a template file"**
     - Haz clic en **"Choose file"** y selecciona tu archivo `infrastructure.yaml`
     - Haz clic en **"Next"**

### Paso 1.2: Configurar parámetros del stack

1. **Configura el nombre del stack**:
   - **Stack name**: `spaceship-registry-dev`
   
2. **Configura los parámetros**:
   - **ProjectName**: `spaceship-registry` (mantener valor por defecto)
   - **Environment**: `dev` (mantener valor por defecto)
   
3. **Continúa**: Haz clic en **"Next"**

### Paso 1.3: Configurar opciones del stack

1. En **Stack options**, deja todas las configuraciones por defecto
2. Haz clic en "Next"

### Paso 1.4: Revisar y crear

1. Revisa toda la configuración
2. En **Capabilities**, marca la casilla "I acknowledge that AWS CloudFormation might create IAM resources"
3. Haz clic en "Submit"

### Paso 1.5: Monitorear el despliegue

1. Observa la pestaña **Events** para ver el progreso del despliegue
2. El proceso toma aproximadamente 3-5 minutos
3. Espera a que el status cambie a **CREATE_COMPLETE**
4. Si hay errores, aparecerán en la pestaña Events con detalles específicos

> ⚠️ **Nota**: El template proveído TIENE errores que se espera puedan ser resueltos por los estudiantes. Son 6 las líneas de infrastructure.yaml que deben ser corregidas para que pueda desplegarse la infraestructura correctamente.

## 2. Configuración inicial de datos

### Paso 2.1: Crear usuarios manualmente

Una vez que el stack esté creado, necesitas crear usuarios en la tabla de usuarios para poder probar el sistema de autenticación.

1. **Accede a DynamoDB**:
   - Ve al servicio **DynamoDB** en la **consola de AWS**
   - En el menú izquierdo, selecciona **"Tables"**
   
2. **Localiza la tabla de usuarios**:
   - Busca y selecciona la tabla que termine con `-users` 
   - Ejemplo: `spaceship-registry-dev-users`
   
3. **Abre el editor de items**:
   - Haz clic en **"Explore table items"**
   - Haz clic en **"Create item"**
   - Si está en "JSON view", cambia a **"Form view"**
   
4. **Crea un usuario**:
   - Agrega los siguientes campos:
     - **username** (String): Tu nombre de usuario personalizado
     - **password** (String): Tu contraseña personalizada
   - Haz clic en **"Create item"**
   
5. **Crea usuarios adicionales**:
   - **Repite los pasos 3-4** para crear algunos usuarios adicionales
   - Usa combinaciones diferentes de username/password
   - **Importante**: Anota las credenciales que crees, las necesitarás más tarde

**Notas importantes:**
- **Seguridad**: Para simplificar el laboratorio, las contraseñas se almacenan en texto plano
- **Producción**: En entornos reales, siempre se deben hashear las contraseñas
- **Recordatorio**: Anota las credenciales exactas que crees; las necesitarás en el **Paso 4.1**

### Paso 2.2: Agregar naves espaciales de ejemplo

Para tener datos de prueba, agrega algunas naves a la tabla de spaceships:

1. **Accede a la tabla de naves**:
   - En **DynamoDB**, selecciona la tabla que termine con `-spaceships`
   - Ejemplo: `spaceship-registry-dev-spaceships`
   
2. **Abre el editor**: Haz clic en **"Explore table items"**

3. **Crea las naves de ejemplo**: Para cada nave, haz clic en **"Create item"** y agrega:

**Nave 1:**
- **spaceship_id** (String): `MILLENNIUM-FALCON`
- **model** (String): `YT-1300`
- **pilot** (String): `Han Solo`

**Nave 2:**
- **spaceship_id** (String): `X-WING-RED5`
- **model** (String): `T-65 X-wing`
- **pilot** (String): `Luke Skywalker`

**Nave 3:**
- **spaceship_id** (String): `SLAVE-I`
- **model** (String): `Firespray-31-class`
- **pilot** (String): `Boba Fett`

### Paso 2.3: Obtener la URL de la API

1. **Accede al stack**:
   - Ve al servicio **CloudFormation** en la **consola de AWS**
   - Selecciona tu stack `spaceship-registry-dev`
   
2. **Obtén la URL de la API**:
   - Ve a la pestaña **"Outputs"**
   - Busca la clave **"ApiUrl"**
   - Copia el valor completo (será similar a):
   ```
   https://abc123def4.execute-api.sa-east-1.amazonaws.com/dev
   ```
   
3. **Guarda la URL**: Anótala en un lugar accesible, la necesitarás en todos los pasos siguientes

### Paso 2.4: Verificar que la API funciona

Antes de implementar la función de login, vamos a probar que las funciones GET funcionan correctamente.

1. **Prueba desde el navegador**:
   - Abre una nueva pestaña en tu navegador web
   - Navega a: `{TU_API_URL}/spaceships` (reemplaza con tu URL real)
   - **Resultado esperado**: JSON con las 3 naves que agregaste en el **Paso 2.2**

2. **Prueba nave específica**:
   - En el navegador, ve a: `{TU_API_URL}/spaceships/MILLENNIUM-FALCON`
   - **Resultado esperado**: JSON solo con los datos del Millennium Falcon

### Paso 2.5: Probar con Postman

Para hacer pruebas más estructuradas:

1. **Abre Postman** (web o aplicación de escritorio)

2. **Crea una nueva colección** llamada "Spaceship Registry CloudFormation API"

3. **Crear request para obtener todas las naves**:
   - **Name**: Get All Spaceships
   - **Method**: GET
   - **URL**: `{TU_API_URL}/spaceships`
   - Ejecuta el request - deberías ver las 3 naves

4. **Crear request para obtener nave específica**:
   - **Name**: Get Spaceship by ID
   - **Method**: GET
   - **URL**: `{TU_API_URL}/spaceships/SLAVE-I`
   - Ejecuta el request - deberías ver solo la nave de Boba Fett

5. **Probar con nave inexistente**:
   - Cambia el ID a algo que no exista: `{TU_API_URL}/spaceships/NONEXISTENT`
   - Deberías recibir un error 404 "Nave no encontrada"

**Checkpoint de validación**: Si estas pruebas funcionan correctamente, significa que:
- Las funciones GET están operativas
- **API Gateway** está configurado correctamente
- Las tablas **DynamoDB** tienen datos válidos
- Puedes proceder con seguridad al **Paso 3** (implementación del login)

## 3. Implementación de la función de login

La función Lambda de login está parcialmente implementada. Debes completar la lógica de acceso a DynamoDB y validación de credenciales.

### Paso 3.1: Analizar el código base

1. **Localiza la función Lambda**:
   - Ve al servicio **Lambda** en la **consola de AWS**
   - Busca y selecciona la función `spaceship-registry-dev-login`
   
2. **Analiza el código base**:
   - Ve a la pestaña **"Code"**
   - Examina el código proporcionado 
   - **Identifica**: Busca los comentarios `# TODO:` que indican qué implementar
   
3. **Estado actual**: La función retorna un error **HTTP 501** "not implemented"

### Paso 3.2: Implementar la lógica de login

Ahora debes implementar la funcionalidad completa de login. Los requisitos son:

1. **Extraer credenciales**: Obtener `username` y `password` del body de la request
2. **Validar entrada**: Verificar que ambos campos estén presentes
3. **Consultar DynamoDB**: Buscar el usuario en la tabla de usuarios
4. **Validar credenciales**: Comparar la password proporcionada con la almacenada
5. **Generar respuesta**: Retornar token JWT si es válido, o error si no lo es

**Consideraciones técnicas importantes**:
- **Función disponible**: Usa `create_jwt_token(username)` que ya está implementada
- **Manejo de errores**: Implementa respuestas apropiadas para:
  - **400**: Datos faltantes o inválidos
  - **401**: Credenciales incorrectas  
  - **500**: Errores internos del servidor
- **Debugging**: Agrega `print()` statements útiles para CloudWatch
- **Variable de entorno**: La tabla está en `os.environ['USERS_TABLE_NAME']`

### Paso 3.3: Configurar permisos de DynamoDB

**¡Importante!** La función Lambda inicialmente NO tiene permisos para acceder a DynamoDB. Debes configurar esto como parte del ejercicio.

1. **Identifica el problema de permisos**:
   - Ejecuta tu función y observa errores como "AccessDeniedException"
   - Los logs de CloudWatch mostrarán exactamente qué permisos faltan

2. **Configura los permisos IAM**:
   - En la función **Lambda** → **"Configuration"** → **"Permissions"**
   - Haz clic en el enlace del **"Role name"** para ir a **IAM**
   - Investiga qué políticas o permisos específicos necesitas agregar

3. **Opciones para agregar permisos** (elige una):
   - **Opción A - Rápida**: Políticas AWS administradas (ej: `AmazonDynamoDBFullAccess`)
     - Ventaja: Implementación inmediata
     - Desventaja: Permisos excesivos
   - **Opción B - Recomendada**: Política personalizada con permisos específicos
     - Ventaja: Principio de menor privilegio
     - Desventaja: Requiere más configuración

4. **Pistas para investigar**:
   - ¿Qué operaciones necesitas realizar en DynamoDB?
   - ¿En qué tabla específica?
   - Busca en la documentación de AWS qué acciones y recursos necesitas

### Paso 3.4: Testing en la consola de Lambda

Para probar tu función directamente en la consola:

1. **Crear evento de prueba**:
   - En la función Lambda, haz clic en "Test"
   - Crea un nuevo evento de prueba llamado `login-test`
   - Usa el siguiente JSON:
   ```json
   {
     "body": "{\"username\": \"admin\", \"password\": \"admin123\"}",
     "headers": {
       "Content-Type": "application/json"
     }
   }
   ```

2. **Ejecutar pruebas**:
   - Haz clic en "Test" para ejecutar
   - Observa la respuesta y los logs en tiempo real
   - Ajusta el código según los errores que encuentres

3. **Probar diferentes escenarios**:
   - Usuario inexistente
   - Password incorrecta
   - JSON malformado
   - Campos faltantes

### Paso 3.5: Debugging con CloudWatch Logs

1. **Acceder a los logs**:
   - Ve a **CloudWatch** en la consola de AWS
   - Selecciona "Log groups" en el menú izquierdo
   - Busca `/aws/lambda/spaceship-registry-dev-login`
   - Haz clic en el log group

2. **Interpretar los logs**:
   - Cada ejecución genera un nuevo log stream
   - Los `print()` en tu código aparecen en los logs
   - Los errores de Python se muestran con stack traces completos

3. **Técnicas de debugging**:
   ```python
   # Agregar logs informativos
   print(f"Evento recibido: {json.dumps(event)}")
   print(f"Buscando usuario: {username}")
   print(f"Usuario encontrado: {'Sí' if 'Item' in response else 'No'}")
   
   # Log de errores detallados
   except Exception as e:
       print(f"Error específico: {type(e).__name__}: {str(e)}")
       print(f"Stack trace: ", exc_info=True)
   ```

## 4. Pruebas de la API con autenticación

### Paso 4.1: Probar el endpoint de login

Una vez que hayas implementado la función de login, utiliza Postman para probarla:

1. En tu colección de Postman, crea un nuevo request para login:
   - **Name**: Login
   - **Method**: POST
   - **URL**: `{TU_API_URL}/login`
   - **Headers**: 
     - `Content-Type: application/json`
   - **Body** (raw, JSON):
     ```json
     {
         "username": "TU_USUARIO",
         "password": "TU_PASSWORD"
     }
     ```
     (**Usa las credenciales exactas** que creaste en el **Paso 2.1**)
     
4. **Envía la request** y espera la respuesta

5. **Resultado esperado**: Respuesta exitosa con un token JWT:
   ```json
   {
       "message": "Login exitoso",
       "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...",
       "user": {
           "username": "TU_USUARIO"
       }
   }
   ```
6. **CRÍTICO**: Copia el valor completo del `token` - **lo necesitarás en el Paso 4.3**

### Paso 4.2: Verificar que GET sigue funcionando sin autenticación

1. **Ejecuta**: Tu request **"Get All Spaceships"** existente
2. **Resultado esperado**: Las mismas 3 naves del **Paso 2.2**
3. **Confirmación**: Las rutas GET están **sin protección**, funcionan sin autenticación

### Paso 4.3: Probar POST de naves espaciales (requiere autenticación)

Ahora probaremos el endpoint protegido:

1. Crea un nuevo request:
   - **Name**: Create Spaceship (Protected)
   - **Method**: POST
   - **URL**: `{TU_API_URL}/spaceships`
   - **Headers**: 
     - `Content-Type: application/json`
     - `Authorization: Bearer {TU_TOKEN_JWT}` (reemplaza {TU_TOKEN_JWT} con el token obtenido del login)
   - **Body** (raw, JSON):
     ```json
     {
         "spaceship_id": "TIE-FIGHTER-01",
         "model": "TIE/ln",
         "pilot": "Imperial Pilot"
     }
     ```
2. **Envía la request**

3. **Resultado esperado**: Respuesta exitosa con **status 201 (Created)**

### Paso 4.4: Verificar que la nueva nave fue creada

1. **Verifica**: Ejecuta nuevamente el request **"Get All Spaceships"**
2. **Resultado esperado**: Ahora deberías ver **4 naves** totales:
   - Las 3 originales del **Paso 2.2**
   - La nueva: **"TIE-FIGHTER-01"**

### Paso 4.5: Probar sin token de autorización

1. Duplica el request anterior
2. Cambia el nombre a "Create Spaceship (No Auth)"
3. Elimina el header `Authorization`
4. Cambia el spaceship_id a algo diferente (ej: `REBEL-TRANSPORT`)
5. **Envía la request**

6. **Resultado esperado**: Error **403 Forbidden**
   - Esto **confirma** que el **Lambda Authorizer** funciona correctamente
   - La protección JWT está **activa** en las rutas POST

### Paso 4.6: Probar búsqueda de la nueva nave

Para completar las pruebas:

1. Crea un nuevo request o modifica el existente:
   - **Name**: Get TIE Fighter
   - **Method**: GET
   - **URL**: `{TU_API_URL}/spaceships/TIE-FIGHTER-01`
2. **Envía la request**

3. **Resultado esperado**: JSON únicamente con los datos de **TIE-FIGHTER-01**

## 5. Análisis del Lambda Authorizer

### Paso 5.1: Entender el flujo de autorización

El Lambda Authorizer implementa el siguiente flujo:

1. **Cliente envía request** con header `Authorization: Bearer <token>`
2. **API Gateway intercepta** el request y extrae el token
3. **Lambda Authorizer recibe** el token y lo valida:
   - Verifica la estructura del JWT
   - Valida la firma usando el secreto compartido
   - Verifica que no haya expirado
4. **Retorna policy** de Allow o Deny
5. **API Gateway permite o deniega** el acceso basado en la policy

### Paso 5.2: Examinar los logs del authorizer

1. Ve a **CloudWatch** en la consola de AWS
2. En el menú izquierdo, selecciona "Log groups"
3. Busca el log group que contenga "authorizer" en el nombre
4. Haz clic en el log group y luego en el log stream más reciente
5. Examina los logs para ver cómo se procesan las requests de autorización

## 6. Limpieza de recursos

### Paso 6.1: Eliminar el stack de CloudFormation

Cuando hayas terminado el laboratorio:

1. Ve a **CloudFormation**
2. Selecciona tu stack `spaceship-registry-dev`
3. Haz clic en "Delete"
4. Confirma la eliminación
5. Monitorea el proceso de eliminación en la pestaña "Events"
6. Verifica que todos los recursos sean eliminados correctamente

**Importante**: CloudFormation eliminará automáticamente todos los recursos creados, incluyendo las tablas DynamoDB y su contenido.

## Conclusiones

En este laboratorio has aprendido:

1. **Infrastructure as Code**: Cómo definir infraestructura compleja usando CloudFormation
2. **Reproducibilidad**: El mismo template puede desplegarse múltiples veces con diferentes parámetros
3. **Autenticación JWT**: Implementación de un sistema de login serverless
4. **Lambda Authorizers**: Protección de APIs usando funciones Lambda personalizadas
5. **Integración de servicios**: Cómo múltiples servicios AWS trabajan juntos en una arquitectura serverless

### Ventajas de Infrastructure as Code con CloudFormation

- **Versionado**: La infraestructura se versiona junto con el código
- **Reproducibilidad**: Fácil recreación en diferentes ambientes
- **Consistencia**: Elimina errores de configuración manual
- **Rollback**: Capacidad de revertir cambios automáticamente
- **Documentación**: El template sirve como documentación de la arquitectura

### Próximos pasos

- Implementar tests automatizados para las funciones Lambda
- Agregar métricas y dashboards personalizados
- Implementar CI/CD para despliegues automáticos
- Explorar AWS SAM (Serverless Application Model) como alternativa
- Integrar con AWS X-Ray para tracing distribuido

¡Felicidades! Has completado exitosamente el Laboratorio 8 y has implementado una infraestructura serverless completa con autenticación JWT usando Infrastructure as Code.

---

## Apéndice: Guía de Troubleshooting

Esta sección consolidada te ayudará a resolver los problemas más comunes que pueden surgir durante el laboratorio.

### A. Problemas de CloudFormation

#### Error: "Template format error"
**Síntoma**: CloudFormation rechaza el template al crearlo
**Posibles causas**:
- Archivo YAML con errores de sintaxis
- Indentación incorrecta
- Caracteres especiales en el archivo

**Solución**:
1. Verifica la sintaxis YAML usando un validador online
2. Asegúrate de que la indentación sea consistente (usa espacios, no tabs)
3. Revisa que no haya caracteres especiales o encoding incorrecto

#### Error: "CREATE_FAILED" en el stack
**Síntoma**: El stack falla durante la creación
**Solución**:
1. Ve a la pestaña "Events" en CloudFormation
2. Busca el recurso que falló y el mensaje de error específico
3. Los errores comunes incluyen:
   - Nombres de recursos duplicados
   - Permisos insuficientes
   - Límites de recursos excedidos

#### Error: "ROLLBACK_COMPLETE"
**Síntoma**: El stack se revierte después de fallar
**Solución**:
1. Elimina el stack fallido completamente
2. Corrige el error identificado en los eventos
3. Vuelve a crear el stack con el template corregido

### B. Problemas de Permisos IAM

#### Error: "AccessDeniedException" en Lambda
**Síntoma**: Función Lambda no puede acceder a DynamoDB
**Causa**: Permisos IAM faltantes o incorrectos

**Solución paso a paso**:
1. **Ve al rol de la función Lambda**:
   - Lambda Console → Function → Configuration → Permissions
   - Haz clic en el enlace del "Role name"

2. **Identifica permisos faltantes**:
   - Revisa los logs de CloudWatch para ver qué acción específica fue denegada
   - Común: `dynamodb:GetItem`, `dynamodb:PutItem`, `dynamodb:Scan`

3. **Agrega permisos usando política administrada**:
   - En IAM, haz clic en "Attach policies"
   - Busca `AmazonDynamoDBFullAccess` (para pruebas)
   - Para producción, crea una política más específica

4. **Crear política personalizada** (recomendado):
   ```json
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Effect": "Allow",
               "Action": [
                   "dynamodb:GetItem",
                   "dynamodb:PutItem",
                   "dynamodb:Scan"
               ],
               "Resource": [
                   "arn:aws:dynamodb:sa-east-1:ACCOUNT-ID:table/spaceship-registry-dev-*"
               ]
           }
       ]
   }
   ```

### C. Problemas de Desarrollo Lambda

#### Error: "Unable to import module 'index'"
**Síntoma**: Lambda no puede ejecutar tu código
**Posibles causas**:
- Función `lambda_handler` mal definida
- Errores de sintaxis Python
- Indentación incorrecta

**Solución**:
1. Verifica que la función se llame exactamente `lambda_handler`
2. Revisa la indentación (Python es sensible a esto)
3. Prueba el código localmente si es posible

#### Error: "KeyError: 'body'" o "'username'"
**Síntoma**: El código no encuentra campos esperados
**Causa**: Estructura del evento diferente a la esperada

**Solución**:
```python
# Verificación defensiva
try:
    body = json.loads(event.get('body', '{}'))
    username = body.get('username')
    password = body.get('password')
    
    if not username or not password:
        return {
            'statusCode': 400,
            'body': json.dumps({'error': 'Username y password requeridos'})
        }
except json.JSONDecodeError:
    return {
        'statusCode': 400,
        'body': json.dumps({'error': 'JSON inválido'})
    }
```

#### Error: "JSONDecodeError"
**Síntoma**: No se puede parsear el JSON del body
**Causa**: Body mal formateado o vacío

**Debugging**:
```python
print(f"Raw body: {event.get('body', 'NO BODY')}")
print(f"Body type: {type(event.get('body'))}")
```

### D. Problemas de DynamoDB

#### Error: "ResourceNotFoundException"
**Síntoma**: No se encuentra la tabla DynamoDB
**Posibles causas**:
- Nombre de tabla incorrecto
- Tabla en región diferente
- Tabla no creada correctamente

**Solución**:
1. **Verifica el nombre de tabla**:
   ```python
   table_name = os.environ.get('USERS_TABLE_NAME', 'NO_DEFINIDA')
   print(f"Usando tabla: {table_name}")
   ```

2. **Verifica que la tabla existe**:
   - Ve a DynamoDB Console
   - Confirma que estás en la región correcta (sa-east-1)
   - Busca las tablas con el prefijo de tu stack

3. **Verifica las variables de entorno**:
   - En Lambda Console → Configuration → Environment variables
   - Debe existir `USERS_TABLE_NAME` y `SPACESHIP_TABLE_NAME`

#### Error: "ValidationException"
**Síntoma**: Error al hacer query/scan a DynamoDB
**Causa**: Uso incorrecto de la clave primaria

**Solución**:
```python
# Para obtener un item por clave primaria
response = table.get_item(
    Key={'username': username}  # 'username' debe ser exactamente el nombre de tu partition key
)

# Para crear un item
table.put_item(
    Item={
        'username': username,
        'password': password
    }
)
```

### E. Problemas de API Gateway y Postman

#### Error: "Internal Server Error" (500)
**Síntoma**: API Gateway retorna error 500
**Causa**: Error en la función Lambda

**Solución**:
1. Ve a CloudWatch Logs de la función Lambda
2. Busca el stack trace completo del error
3. Los errores 500 generalmente indican problemas de código, no configuración

#### Error: "Forbidden" (403) inesperado
**Síntoma**: Endpoint que debería funcionar retorna 403
**Posibles causas**:
- Token JWT malformado o expirado
- Lambda Authorizer fallando
- CORS mal configurado

**Debugging**:
1. **Verifica el token JWT**:
   - Cópialo y pégalo en [jwt.io](https://jwt.io) para verificar formato
   - Revisa que no haya espacios extra al copiarlo

2. **Revisa logs del Lambda Authorizer**:
   - CloudWatch → Log groups → buscar "authorizer"
   - Debe mostrar si el token es válido o no

#### Error: "CORS" en navegador
**Síntoma**: Requests fallan por política CORS
**Causa**: Headers CORS no configurados correctamente

**Verificación**:
- El template ya incluye configuración CORS
- Verifica que las funciones Lambda retornen headers CORS:
```python
'headers': {
    'Content-Type': 'application/json',
    'Access-Control-Allow-Origin': '*'
}
```

### F. Metodología General de Debugging

#### Paso 1: Identificar el componente que falla
- **CloudFormation**: Error durante despliegue
- **Lambda**: Error 500 desde la API
- **API Gateway**: Error 4xx desde la API
- **DynamoDB**: Error específico en logs
- **IAM**: AccessDeniedException

#### Paso 2: Revisar logs específicos
- **CloudFormation**: Pestaña Events del stack
- **Lambda**: CloudWatch Log Groups `/aws/lambda/FUNCTION_NAME`
- **API Gateway**: CloudWatch Log Groups si están habilitados

#### Paso 3: Usar debugging sistemático
```python
# En las funciones Lambda, agrega logs detallados
import json

def lambda_handler(event, context):
    print(f"=== DEBUG START ===")
    print(f"Event: {json.dumps(event, indent=2)}")
    print(f"Context: {context}")
    
    try:
        # Tu código aquí
        pass
    except Exception as e:
        print(f"ERROR: {type(e).__name__}: {str(e)}")
        import traceback
        traceback.print_exc()
        raise
    
    print(f"=== DEBUG END ===")
```

#### Paso 4: Probar componentes individualmente
1. **Primero**: Prueba Lambda directamente (console test)
2. **Segundo**: Prueba API Gateway sin authorizer
3. **Tercero**: Agrega authorizer y prueba flujo completo
