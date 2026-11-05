# Validación

Fecha: 2026-11-05.

- GCC MinGW 6.3: compilación C11 con -Wall -Wextra -Werror -O2, sin advertencias.
- verificar-windows.ps1: 20/20 comprobaciones contra archivos dorados.
- Harness pruebas/dce.c: elimina cadena de temporales muertos y conserva la viva.
- Validación Linux x86-64, sanitizadores y ELF: pendiente de ejecución en CI.
- Los archivos originales de tests/ se conservan; las pruebas adicionales viven
  en pruebas/ y en tests/propias/.

Reproducir en Linux: make clean, make, make test y make demo-bin.
La salida de Windows no acredita un ELF ni una demo realizada por el equipo.
