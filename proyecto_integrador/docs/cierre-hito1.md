# Cierre de la entrega del Hito 1

## Comprobaciones

- [ ] El equipo revisó el lexer, parser, AST y comprobaciones S01/E01.
- [x] Windows mostró 4/4 oficiales y 8/8 propias en el verificador nuevo; captura conservada en `evidencias/01-pruebas.png`.
- [x] Linux mostró 4/4 en `make test-hito1`; captura conservada en `evidencias/09-linux-hito1.png`. Los ocho casos propios se acreditaron en Windows.
- [ ] Se anexaron las salidas reales con fecha y commit.
- [ ] El reporte se revisó y se completaron docente y grupo si hacen falta.
- [x] README describe qué funciona y el alcance de la entrega.
- [ ] La bitácora registra decisiones y auditoría humana reales.
- [ ] Los commits reflejan aportaciones reales de los integrantes.
- [ ] Se ensayó la demo de tres minutos.
- [x] Tag `v0.1-hito1` existente verificado sobre `3e09970377805c528cccd4936de765530bf96d00`; no se movió.

## Integración y tag

El ZIP es un paquete de preparación, no sustituye el fork del curso. Integrar `proyecto_integrador/` en la copia Git existente; conservar el historial y todos los archivos ajenos al hito. Comparar los cambios antes de subir. `tests/` debe quedar sin cambios.

En una terminal del repositorio, revisar:

```text
git status --short
git diff -- proyecto_integrador/tests
git diff --stat
git tag --list v0.1-hito1
```

Después de guardar las aportaciones reales en un commit y ejecutar las pruebas sobre ese estado, crear el tag si todavía no existe:

```text
git tag -a v0.1-hito1 -m "Entrega del Hito 1"
git push origin HEAD
git push origin v0.1-hito1
```

Si el tag ya existe, revisar a qué commit apunta; no borrarlo ni forzarlo. Estos comandos publican cambios: deben ejecutarse sobre el repositorio y la rama elegidos por el equipo. No se ejecutaron automáticamente. Al revisar GitHub se comprobó que el tag ya existe; no debe recrearse ni moverse sin una decisión expresa del equipo. Al revisar GitHub se comprobó que el tag ya existe; no debe recrearse ni moverse sin una decisión expresa del equipo.

## Evidencia conservada

`evidencia-windows-previa.png` es la captura aportada por el usuario con GCC 6.3.0 y 20/20 pruebas generales. `verificacion-del-paquete.txt` registra comprobaciones de preparación del paquete, no una ejecución del compilador. El reporte separa ambas cosas de las pruebas pendientes.
