¿puedes hacerme un resumen de requisitos para cumplir con la nueva normativa de control de

asistencia de los trbajadores?

actualmente, usamos odoo 14ce, sin recoger información de ubicación.

¡Claro! La normativa de control de asistencia en España se basa en el Real Decreto-ley 8/2019, que modifica el Estatuto de los Trabajadores. El requisito principal es que la empresa garantice el registro diario de la jornada de cada trabajador.

Dado que están usando Odoo 14 CE sin recoger ubicación, estos son los requisitos esenciales que deben cumplir y cómo pueden abordarlos:

📅 Requisitos Esenciales de la Normativa
Requisito	Descripción Legal	Implicación para Odoo 14
Registro Diario Obligatorio	Registrar el inicio y la finalización de la jornada de cada trabajador, de forma fehaciente.	Su módulo de asistencia debe registrar al menos la hora de fichaje de entrada y la hora de fichaje de salida (y cualquier pausa relevante).
Jornada Completa	El registro debe incluir el tiempo efectivo de trabajo.	Deben tener un sistema que asegure que el empleado solo pueda fichar dentro del día laboral y que no omita el registro de salida.
Disponibilidad para Inspección	Los registros deben estar accesibles a los trabajadores, sus representantes sindicales y a la Inspección de Trabajo.	Los datos deben estar almacenados de forma inalterable y ser exportables (PDF, Excel) de manera rápida.
Conservación	Los registros deben conservarse durante cuatro años.	Deben asegurar la copia de seguridad y la retención de los datos históricos en Odoo.
Información al Trabajador	La empresa debe notificar a los trabajadores sus condiciones de jornada y el sistema de registro de forma clara.	Documentación interna y comunicados que describan cómo funciona el módulo de Odoo.
⚙️ Integración con Odoo 14 Community Edition (CE)
El módulo base de Fichaje (Time Off) u otros módulos custom de asistencia en Odoo 14 CE pueden gestionar el registro, pero deben asegurarse de lo siguiente:

1. Integridad y Trazabilidad de los Datos
Imposibilidad de Modificación: El sistema debe ser diseñado para que los usuarios (empleados) no puedan modificar sus fichajes de entrada/salida una vez realizados. Solo los administradores o RRHH deben poder corregir errores, y esas correcciones deben quedar registradas (trazabilidad).

Automatización: Es preferible usar un reloj fichador (Kiosk Mode en Odoo si lo tienen implementado) o un sistema web/móvil simple para que el registro sea automático al inicio y fin de la sesión, minimizando errores.

2. Exportación de Registros
Deben crear un informe que permita exportar los datos de asistencia por trabajador y por rango de fechas, incluyendo:

Nombre del Empleado

Fecha

Hora de Entrada (Check-in)

Hora de Salida (Check-out)

Tiempo Efectivo Trabajado

3. Ausencia de Ubicación
Dado que han optado por no recoger la ubicación del fichaje:

Deben tener claramente documentado que el fichaje se realiza dentro de las instalaciones de la empresa (si es el caso) o a través de una red corporativa, para evitar ambigüedades en caso de inspección.

Si tienen teletrabajo, es importante tener un acuerdo por escrito que valide el sistema de registro (Odoo) como el método fehaciente.

⚠️ Aspecto de la No-Ubicación
La normativa no exige la geolocalización, pero sí la fehaciencia. Si el fichaje se realiza desde dispositivos personales fuera de las instalaciones sin control de IP o de red, puede ser más difícil demostrar la fehaciencia del inicio de la jornada en caso de litigio, aunque el requisito legal se cumple al registrar la hora.

un aspecto que no tengo cubierto es el de la inalterabilidad. los datos están disponibles como en cualquier otro modelo del sistema, pudiendo modificarse con los permisos adecuados

Es una preocupación crítica y muy válida. La inalterabilidad es el pilar de la fehaciencia exigida por la Inspección de Trabajo.

En Odoo, donde los datos de la asistencia (hr.attendance) se almacenan como cualquier otro registro, la inalterabilidad no es nativa, pero se puede lograr mediante una combinación de configuración, trazabilidad y un pequeño cambio técnico.

Aquí tienes las estrategias, de la más simple a la más robusta, para asegurar la inalterabilidad de los registros de fichaje en Odoo 14 CE:

1. 🛡️ Control de Permisos (La Primera Línea de Defensa)
La forma más rápida de limitar la modificación es asegurarse de que solo los usuarios estrictamente necesarios puedan escribir en el modelo hr.attendance.

Revisa las Reglas de Acceso:

Ve a Ajustes > Técnico > Seguridad > Reglas de Acceso.

Busca el modelo Asistencia (hr.attendance).

Asegúrate de que los grupos de usuarios (excepto quizás el Administrador y un rol específico de RRHH/Nóminas) no tengan marcado el permiso de Escritura o Borrado.

Regla de Seguridad de Registro (Record Rule):

Asegúrate de que no haya una regla de registro que permita al propio empleado editar sus fichajes pasados. Una regla de este tipo debe aplicar solo a los fichajes en borrador o en el mismo momento de la creación.

2. 📝 Trazabilidad de Auditoría (La Prueba de Modificación)
Si bien la modificación puede ocurrir, la inalterabilidad se demuestra si todo cambio queda registrado y es visible para el auditor. Odoo lo hace automáticamente con el Registro de Actividad.

Habilitar el Seguimiento de Campos Clave
Activa el Modo Desarrollador.

Ve al Modelo hr.attendance:

Ve a Ajustes > Técnico > Estructura de Base de Datos > Modelos.

Busca hr.attendance.

Asegúrate de que el campo "Seguimiento de mensajes" (mail.thread) esté activo y que los campos clave (check_in, check_out) estén configurados para seguirse.

Resultado: Cualquier modificación en la hora de entrada o salida (hecha por cualquier usuario, incluido el administrador) se registrará automáticamente en el historial del registro (pestaña "Registro de Actividad"), mostrando:

Quién lo modificó.

Cuándo lo modificó.

Cuál era el valor antiguo y el valor nuevo.

Esto prueba que, aunque el dato se haya modificado, la empresa puede rastrear y justificar el cambio.

3. 🔐 Inalterabilidad Técnica (El Sello Fechaciente)
Esta es la solución más robusta y es la que más se acerca al concepto de "sello digital" o hash que exige la normativa. Requiere una pequeña modificación de código en su módulo custom o un parche de su módulo de asistencia.

El principio es: una vez que un fichaje está completo, se "sella" y ya no puede cambiarse.

Implementar un Campo de Sello (Hash)
Añade un Campo No Editable: Añade un campo Char al modelo hr.attendance llamado, por ejemplo, registro_sellado.

Calcula y Guarda un Hash: Sobrescribe el método write() o create() del modelo hr.attendance. Cuando un registro de asistencia pase a un estado "Completo" (es decir, ya tiene check_in y check_out), el sistema debe:

Concatenar las horas de check_in, check_out y el ID del empleado.

Calcular un hash criptográfico (ej. SHA-256) de esta cadena.

Guardar ese hash en el campo registro_sellado.

Bloqueo: Añade una restricción (constraint) o una regla de código: si el campo registro_sellado no está vacío, nadie puede modificar los campos check_in o check_out.

El Sello: El valor del registro_sellado se convierte en la prueba inalterable del momento del fichaje. Si alguien modifica check_in en la base de datos, el hash almacenado ya no coincidirá con el hash recalculado, invalidando el registro ante una inspección.

Si implementan los pasos 1 y 2, ya tienen un argumento sólido para demostrar la fehaciencia ante la Inspección de Trabajo. La solución técnica (paso 3) es el nivel más alto de cumplimiento.

sobre el punto 1, creo que no se puede implementar ya que el diseño hace que el fichaje de salida se registre "actualizando" el registro "creado" por el de entrada. Por tanto es necesario que el usuario tenga permiso de creación y modificacion

Entendido. Este es un punto de conflicto de diseño fundamental en Odoo y tiene toda la razón: si el empleado necesita permiso de Escritura para registrar la salida (check_out), ese mismo permiso le permite modificar la entrada (check_in) o cualquier otro registro pasado.

La inalterabilidad para registros completados se resuelve en Odoo separando la capacidad de modificar el registro en curso de la capacidad de modificar un registro terminado.

La solución más limpia y que no requiere tocar el código (solo configuración) es mediante una Regla de Registro (ir.rule) estricta, aplicada al modelo hr.attendance.

🔐 Solución: Regla de Registro por Estado
Debe crear una regla de registro que restrinja el permiso de escritura del grupo de empleados al modelo hr.attendance basándose en si el registro está completado o no.

1. Crear la Regla de Bloqueo (Inalterabilidad)
Esta regla debe aplicarse a los grupos que tienen permiso de escritura (por ejemplo, el grupo Employee) y debe impedirles escribir si el registro ya tiene una hora de salida.

Campo	Valor	Descripción
Objeto	Asistencia (hr.attendance)	
Grupo	Employee (o el grupo con permisos de escritura)	
Permisos	Escritura: Desmarcar	Esta regla se aplicará al permiso de escritura.
Dominio de Regla	[('check_out', '!=', False)]	Aplica la restricción a todos los registros que ya tienen una salida (completados).
Resultado	El empleado no puede modificar ningún campo de los registros donde ya existe check_out.	
2. Crear la Regla de Permiso (Permitir Salida)
Esta regla es complementaria y asegura que el empleado SÍ puede actualizar el registro de la sesión actual para marcar la salida.

Campo	Valor	Descripción
Objeto	Asistencia (hr.attendance)	
Grupo	Employee (o el grupo con permisos de escritura)	
Permisos	Escritura: Marcar	Esta regla anula el bloqueo anterior y otorga el permiso de escritura.
Dominio de Regla	[('check_out', '=', False), ('user_id', '=', user.id)]	Permite escribir solo en el registro actual (no completado) y solo si pertenece al usuario.
Resultado	El empleado puede establecer la hora check_out en el registro abierto, pero no en ningún otro registro.	
🎯 Funcionamiento Garantizado
Al aplicar estas dos reglas simultáneamente, se logra la inalterabilidad:

Registro Abierto: El registro cumple el dominio de la Regla 2 (check_out = False), permitiendo al empleado actualizarlo y registrar la salida.

Registro Cerrado: Una vez que el check_out se registra, el registro pasa a cumplir el dominio de la Regla 1 (check_out != False), y pierde instantáneamente el permiso de escritura, quedando sellado para ese usuario.

Esto cumple con la inalterabilidad de los registros históricos de forma nativa en la configuración de Odoo.

esto haría innecesario configurar el punto 3, el hash?

Sí, haría innecesario implementar el hash (punto 3) para cumplir con el requisito de inalterabilidad ante la Inspección de Trabajo.

🔒 Inalterabilidad por Regla de Acceso
La solución de las Reglas de Registro (ir.rule) logra la inalterabilidad a nivel de la aplicación (Odoo), que es el objetivo de la normativa española.

¿Por qué es suficiente?
Bloqueo Funcional: La Regla de Registro elimina el permiso de Escritura a los usuarios (empleados) en el momento exacto en que el registro de asistencia se completa (check_out != False). Esto significa que, a nivel de la interfaz de usuario y la lógica de negocio de Odoo, el dato está sellado.

Trazabilidad: Al combinar esto con la Auditoría (punto 2), si un administrador o un usuario con permisos superiores modifica el registro, esa modificación queda grabada con el nombre del usuario y la hora del cambio.

Cumplimiento Legal: La normativa exige que el sistema sea "fehaciente" (digno de fe y fiable). Un sistema que bloquea la edición por Regla de Registro y audita cualquier cambio realizado por Administrador cumple con la obligación de control y trazabilidad.

El hash (punto 3) solo sería necesario si necesitaran una prueba de inalterabilidad a nivel de base de datos (por ejemplo, para probar que el propio administrador de la base de datos no modificó el registro sin dejar rastro en el log de Odoo), pero esto excede lo que la mayoría de las inspecciones de trabajo exigen en España.

En resumen: Con los puntos 1 y 2 bien implementados (Regla de Bloqueo + Auditoría), el sistema es considerado fehaciente y el hash es un gasto de desarrollo innecesario.

sobre la trazabilidad de auditoría, no puedo habilitar vía ui las opciones indicadas, tendré que hacerlo con herencia del modelo indicando "tracking" en los campos necesarios

Efectivamente. En Odoo 14 CE, si la interfaz de usuario no te permite configurar el seguimiento de campos clave, la forma correcta y estándar de habilitar la trazabilidad de auditoría (tracking) es mediante una herencia del modelo Python y la adición del atributo tracking=True a las definiciones de los campos.

No es necesario sobrescribir el método write() ni la lógica, solo heredar la clase para añadir esta configuración.

💻 Pasos para Habilitar el Tracking en hr.attendance
Para garantizar que cada cambio en la hora de entrada (check_in) y la hora de salida (check_out) quede registrado en el historial (Registro de Actividad/Chatter), debes heredar el modelo hr.attendance en tu módulo custom.

1. Modificar la Clase Python
En tu archivo Python (models/attendance.py o similar) de tu módulo custom, hereda el modelo y redefine los campos check_in y check_out para incluir el parámetro tracking=True.

Python
from odoo import fields, models

class HrAttendance(models.Model):
    _inherit = 'hr.attendance'
    
    # Heredamos los campos y añadimos tracking=True
    check_in = fields.Datetime(tracking=True) 
    check_out = fields.Datetime(tracking=True)
    
    # Opcional: Si quieres registrar los cambios en la ubicación si la usáis
    # check_in_latitude = fields.Float(tracking=True)
    # check_out_latitude = fields.Float(tracking=True)
2. Verificar la Herencia de la Clase mail.thread
Para que el tracking=True funcione, el modelo debe heredar la clase de seguimiento de mensajes (mail.thread). El modelo hr.attendance ya lo hace en Odoo por defecto, pero es bueno confirmarlo:

Python
class HrAttendance(models.Model):
    _inherit = ['hr.attendance', 'mail.thread'] # Asegura que mail.thread está presente
    # ...
3. Actualizar el Módulo
Después de realizar el cambio en el código Python, debes actualizar tu módulo custom para que Odoo recoja la nueva definición de los campos.

✅ Resultado
Una vez actualizado, cualquier usuario que modifique manualmente la hora de Entrada o Salida en el formulario de asistencia provocará una entrada automática en el Chatter, indicando: quién hizo el cambio, cuándo y cuál era el valor antiguo y el nuevo. Esto proporciona la trazabilidad de auditoría requerida para justificar cualquier corrección de fichaje.





