# Aplicación Web Móvil para Administración de Finanzas Personales

Esta aplicación web fue desarrollada con **HTML**, **CSS** y **JavaScript**, utilizando el modelo **ChatGPT o3-mini** como asistente en su diseño. Está pensada para ayudarte a gestionar tus finanzas personales de forma sencilla desde tu dispositivo móvil.

## Funcionalidades

- Gestión de **tarjetas** y **cuentas de crédito**.
- Control automático de **fechas de corte** y **fechas de pago**.
- Registro de **transacciones** con cálculo automático de la próxima fecha de pago según la fecha de corte.
- Información clara y organizada sobre pagos:
  - **Mensuales**
  - **Semanales**
  - **Quincenales**

## Enlace a la aplicación

[https://glitchrob.github.io/index.html](https://glitchrob.github.io/index.html)

Gestor Financiero Personal

Sitio en vivo: https://glitchrob.github.io/index.html

Este proyecto es una herramienta web para la gestión de cuentas y finanzas personales o familiares. Permite registrar ingresos, gastos, fechas clave y realizar un seguimiento detallado de deudas y pagos.


---

Características

Gestión de múltiples cuentas con nombres, colores y fechas personalizadas.

Registro de ingresos y egresos con detalles como monto, cantidad, tipo de pago y descripción.

Cálculos automáticos de pagos mensuales, totales por periodo, y deuda global.

Interfaz dinámica e interactiva que se actualiza en tiempo real.

Almacenamiento local en el navegador para mantener la privacidad de los datos.



---

Tecnologías Utilizadas

HTML – Estructura principal del sitio.

CSS – Estilos personalizados para la interfaz de usuario.

JavaScript – Lógica de interacción, cálculos financieros y manejo de datos.



---

Funcionamiento Técnico

Estructura General

La aplicación está completamente basada en el navegador, por lo que no requiere instalación ni backend. Todo se ejecuta localmente, utilizando:

localStorage para guardar cuentas, transacciones y configuración del usuario.

Scripts JavaScript que realizan cálculos automáticos según la fecha actual y los datos ingresados.

Módulos interactivos (modales) para añadir, editar y ver detalles de las cuentas y movimientos.


Cálculos y Automatización

Determina fechas futuras de corte y pago basadas en fechas configuradas por el usuario.

Calcula montos a pagar por semana (viernes) o por mes dependiendo del tipo de gasto.

Ajusta automáticamente las fechas si la actual ya pasó el periodo de pago.

Presenta los totales por categoría y por cuenta, facilitando el control de finanzas.


Privacidad

No hay conexión a servidores externos: los datos se guardan localmente en el navegador.

No se requiere autenticación o creación de cuenta.

Si se borra el caché del navegador, los datos se perderán.



---

Limitaciones

Los datos no se sincronizan entre dispositivos.

La información se pierde si se borra el almacenamiento local del navegador.



---

Uso

1. Abre https://glitchrob.github.io/index.html


2. Agrega cuentas y configura fechas de corte y pago.


3. Registra tus movimientos (ingresos o egresos).


4. Observa tus cálculos automáticos y mantén el control de tus finanzas.




---

Contribuciones

Este proyecto puede expandirse para incluir:

Exportación/importación de datos.

Sincronización en la nube.

Notificaciones o recordatorios.


Si deseas contribuir, puedes hacer un fork y enviar un pull request.
