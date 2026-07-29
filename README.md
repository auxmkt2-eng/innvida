# INNVIDA CEO Command Center

Dashboard independiente para dirección: conecta en tiempo real las cotizaciones de Sanaré y Nomad, convierte estatus en cierre estimado, identifica riesgos accionables y exporta el reporte semanal del CEO en Excel.

## Arranque local

1. Copie `firebase-config.example.js` como `firebase-config.js`.
2. Capture mediante un canal seguro los valores de Firebase de cada proyecto. El archivo real está ignorado por Git y no debe compartirse.
3. Sirva esta carpeta con un servidor estático. Por ejemplo, desde una terminal con Node instalado: `npx serve .`.
4. Abra la dirección local mostrada por el servidor.

No hay datos de muestra: antes de configurar Firebase, el tablero muestra explícitamente que no existe un corte disponible.

## Modelo detectado en la referencia

Ambos proyectos usan Firestore, colección `cotizaciones`. Campos comunes: `folio`, `fechaEmision`, `fechaCierre`, `paciente`, `medico`, `kam`, `aseguradora`, `total`, `status1`, `status2`, `motivo`, `createdAt`. Sanaré incorpora `telefono`, `direccion`, `dx`, `esquema`, `servicios`, `medicamentos`; Nomad incorpora `marca`, `diagnostico`, `pruebas`. Sanaré deriva sede de dos teléfonos conocidos; el Command Center no asigna una sede cuando no hay una regla confiable.

### Inconsistencias a resolver con operación

- `fechaEmision` y `fechaCierre` se manejan como texto ISO; registros con otros formatos no pueden ordenarse con precisión.
- `proximaAccion` no existe de forma consistente en la fuente. El tablero lo lee y lo guarda bajo ese nombre cuando las reglas lo permiten.
- No existe una meta mensual observada. Las metas de junio se capturan/importan localmente por dispositivo hasta que se conecte una colección protegida de metas.
- “Médico recurrente” se calcula por coincidencia exacta de nombre; conviene agregar un ID médico canónico.

## Operación

- La ponderación es: Cerrada/Aceptada 100%, En negociación 60%, Cotización enviada 35%, Sin seguimiento 15%, Perdida/Rechazada/Cancelada 0%.
- En el detalle, `Guardar` actualiza `status1`, `motivo`, `fechaCierre` y `proximaAccion` en Firestore. Si las reglas no autorizan escribir, muestra el error sin modificar el tablero.
- “Generar reporte CEO” crea un `.xlsx` filtrado con resumen ejecutivo, una hoja por KAM y hallazgos/prioridades.
- Para el reporte de viernes, seleccione el periodo y genere el archivo antes de las 12:00 pm. Para el lunes, filtre cada KAM y use los campos motivo/próxima acción en los casos no cerrados.

## Seguridad y despliegue

No hay contraseñas ni usuarios hardcodeados. Use Firebase Authentication o el control de acceso de su proveedor de hosting y reglas Firestore por rol antes de desplegar. Una clave pública de configuración Firebase no es un secreto por sí misma, pero este repositorio la separa para no exponer configuraciones operativas. Para un despliegue ejecutivo, hospede detrás de SSO y permita actualización solo a roles autorizados.

Las reglas recomendadas deben limitar lectura a usuarios autenticados autorizados y escritura únicamente a roles KAM/operación; la validación de campos debe ocurrir en reglas o backend. Nunca dependa de la interfaz para controlar el acceso.
