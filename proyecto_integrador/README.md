# Proyecto Integrador · Lenguajito

Equipo: **Ingrid Paola Dzul Segovia, Jonathan Uriel Che Castillo y Josue Efrain Martin Pool**.

Compilador en C11 para la [gramática del curso](docs/gramatica.md). Lee programas
`.lj` y ofrece `--tokens`, `--ast`, `--check`, `--tac`, `--asm` y `--run`.
Sin modo realiza la comprobación semántica. No requiere bibliotecas de terceros.

## Ejecutar en Linux x86-64 (entrega oficial)

Requiere GCC, GNU as, make y Python 3.10 o posterior; para inspeccionar el ELF,
file o readelf. Se puede usar una terminal Linux o un Codespace del repositorio.

```bash
cd proyecto_integrador
make
make test-hito1
make test-hito2
make test-hito3
make test-hito4
make test
make demo-bin
readelf -h build/anidado
```

`make test` conserva las 20 pruebas doradas y agrega pruebas propias y un
harness de eliminación de temporales muertos. `make demo-bin` deja el ELF
`build/anidado` y su ensamblador para la entrega.

```bash
./lenguajito tests/validos/08_anidado.lj --tokens
./lenguajito tests/validos/08_anidado.lj --ast
./lenguajito tests/invalidos/02_redeclaracion.lj --check
./lenguajito tests/tac/01_folding.lj --tac
./lenguajito tests/validos/08_anidado.lj --asm
./lenguajito tests/validos/08_anidado.lj --run
```

## Ejecutar en este Windows

En PowerShell, desde esta carpeta:

```powershell
.\verificar-windows.ps1
$env:PATH = 'C:\MinGW\bin;' + $env:PATH
.\lenguajito.exe tests\validos\08_anidado.lj --run
```

La compatibilidad local de `--run` usa MinGW **de 32 bits** y ejecuta un PE.
`--asm` siempre produce la salida oficial Linux x86-64 AT&T.
Un PE de Windows no sustituye al ELF requerido en el Hito 4.
No se necesita modificar `src/Main.java`: este trabajo se desarrolla en C.

## Qué funciona

- Tokens con posiciones, comentarios, categorías y EOF.
- Parser descendente escrito a mano, AST y gramática completa.
- Ámbitos, shadowing, redeclaraciones y lectura antes de asignación.
- Inicialización definida: en un if/else se intersectan los estados; un while
  puede no ejecutarse, por lo que no inicializa variables después del ciclo.
- Diagnósticos S01/E01/E02/E04 con un máximo de 20.
- TAC con etiquetas y temporales únicos, plegado y eliminación de temporales muertos.
- Backend con casillas independientes para variables ocultas por shadowing.
- Ensamblado, enlace y ejecución de un binario real.
- Aritmética de 32 bits con envoltura; división entera trunca hacia cero.

## Límites y pendientes de entrega

El resultado de pruebas está en [validación](docs/validacion.md).
No se exige diagnosticar división dinámica por cero.
Los miembros deben auditar el código, registrar sus aportaciones reales,
realizar los avances y presentar la demo. Esta implementación asistida por IA
no acredita por sí sola participación individual ni presentaciones en clase.
Los tags se crean cuando el equipo acepta cada entrega; no se inventa una
cronología de cuatro hitos realizados en distintas sesiones.

## Reparto propuesto para la revisión del equipo

| Integrante | Archivos que debe revisar, explicar y completar | Evidencia individual |
| :--- | :--- | :--- |
| Ingrid Paola Dzul Segovia | `lexer.c`, `parser.c`, `ast.c` | Pendiente de revisión y commits propios |
| Jonathan Uriel Che Castillo | `semantica.c`, `tac.c`, `util.c` | Pendiente de revisión y commits propios |
| Josue Efrain Martin Pool | `codegen.c`, integración, pruebas y documentación | Pendiente de revisión y commits propios |

Este reparto es una propuesta, no una afirmación de autoría humana del código
generado durante esta conversación.

## Documentación

- [Diseño](docs/diseno.md) y [optimización](docs/optimizacion.md).
- [Guion de demo](docs/demo.md), [avances](docs/avances.md).
- Entregas: [Hito 1](docs/entrega_hito1.md), [Hito 2](docs/entrega_hito2.md),
  [Hito 3](docs/entrega_hito3.md), [Hito 4](docs/entrega_hito4.md).

