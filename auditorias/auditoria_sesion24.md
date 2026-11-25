
# Bitácora de Requerimientos y Auditoría IA — Sesión 24
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Control de Flujo: Sentencia if-else  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Josue Efrain Martin Pool  
**Matrícula:** 24531353        
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Saltos condicionales y evaluación booleana  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*

> Al realizar la función manualmente, tuve que identificar cómo generar los saltos dependiendo del operador lógico. La parte que más se me complicó fue diferenciar el comportamiento de AND (`&`) y OR (`|`), porque AND necesita saltar cuando un operando es falso, mientras que OR debe saltar cuando uno es verdadero. También tuve que considerar qué hacer cuando el segundo operando es NULL, para generar solamente una instrucción. Otro detalle fue controlar la cantidad de líneas disponibles mediante `max`, evitando escribir más instrucciones de las permitidas. Mi implementación manual utilizó `sprintf`, por lo que no controla directamente el tamaño de cada línea.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar. Se debe respetar la interfaz declarada en `api_sesion24.h`. |
| **Rol (R)** | Ingeniero de software especializado en sistemas y compiladores. |
| **Especificación (E)** | Implementar `emitir_condicion()` para generar saltos condicionales. Para AND (`&`) se utiliza `IFFALSE`; para OR (`|`) se utiliza `IFTRUE`. Ambos saltos deben dirigirse a la etiqueta recibida. Si `b` es NULL, solamente se genera un salto. Se deben respetar los límites de `max` y validar los argumentos necesarios. |
| **Output (O)** | Código modular en C, sin función `main()`, compatible con el harness y compilable con `-Wall -Wextra -Werror`, además de un análisis de complejidad. |

### 2.2 Prompt Estructurado

```text
Contexto:
Estoy trabajando en la Sesión 24 de Lenguajes y Autómatas II.
Necesito implementar una función en C para generar saltos
condicionales de código intermedio, respetando el contrato
de api_sesion24.h.

Rol:
Actúa como un ingeniero de software especializado en
compiladores y generación de código intermedio.

Especificación:
Implementa exactamente la siguiente función:

int emitir_condicion(char lineas[][64], int max,
                     const char *a, const char *b,
                     char op, const char *etiqueta);

Requisitos:
1. Si el operador es '&' (AND), genera instrucciones
   con el formato:
   IFFALSE operando GOTO etiqueta

2. Si el operador es '|' (OR), genera instrucciones
   con el formato:
   IFTRUE operando GOTO etiqueta

3. Debe generar primero el salto del operando a y después
   el del operando b, cuando este exista.

4. Si b es NULL, debe generar solamente el salto
   correspondiente al operando a.

5. Debe respetar el límite max y nunca escribir más
   instrucciones de las que permite el arreglo.

6. Cada línea tiene una capacidad máxima de 64 caracteres.

7. Si max es menor o igual a cero, o los argumentos
   obligatorios son NULL, debe regresar 0.

8. Debe regresar la cantidad de instrucciones generadas.

9. No debe utilizar memoria dinámica ni variables globales.

10. No debe incluir main(), porque el harness ya contiene
    su propia función principal.

Output:
Devuelve únicamente el código completo de ia_sesion24_c.c,
con comentarios claros y utilizando snprintf para controlar
el tamaño de las cadenas. Incluye una breve explicación
de la complejidad temporal y espacial.
Respeta exactamente la firma de la función y asegúrate
de que compile con GCC usando:
-std=c11 -Wall -Wextra -Werror
```

---

## 3. Código Devuelto por el Modelo

```c
/* ia_sesion24_c.c - Fase 2: El Cientifico
 *
 * Solución robusta y optimizada por IA.
 */
#include "api_sesion24.h"
#include <stdio.h>
#include <string.h>

int emitir_condicion(char lineas[][64], int max, const char *a, const char *b, char op, const char *etiqueta) {
    if (max <= 0 || a == NULL || etiqueta == NULL) {
        return 0;
    }

    int contador = 0;
    const char *tipo_salto = (op == '|') ? "IFTRUE" : "IFFALSE";

    // Primer operando
    snprintf(lineas[contador], 64, "%s %s GOTO %s", tipo_salto, a, etiqueta);
    contador++;

    // Segundo operando (si existe y hay espacio suficiente)
    if (b != NULL && contador < max) {
        snprintf(lineas[contador], 64, "%s %s GOTO %s", tipo_salto, b, etiqueta);
        contador++;
    }

    return contador;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)

*Pruebas ejecutadas:* Condiciones compuestas con AND y OR, y una condición simple con segundo operando NULL.

### Salida del harness

**Pruebas de la implementación manual y de la implementación de IA: ambas obtuvieron 6/6.**

```text
=== AUDITORIA SESION 24 - Saltos condicionales con cortocircuito ===
Caso                               esperado:              obtenido:              resultado
[ 1] a && b (primer salto)
     esperado: IFFALSE a GOTO L9
     obtenido: IFFALSE a GOTO L9
     -> OK
[ 2] a && b (segundo salto)
     esperado: IFFALSE b GOTO L9
     obtenido: IFFALSE b GOTO L9
     -> OK
[ 3] a && b (dos saltos)                esperado: 2            obtenido: 2            OK
[ 4] a || b (primer salto)
     esperado: IFTRUE a GOTO L9
     obtenido: IFTRUE a GOTO L9
     -> OK
[ 5] a || b (segundo salto)
     esperado: IFTRUE b GOTO L9
     obtenido: IFTRUE b GOTO L9
     -> OK
[ 6] condicion simple (b NULL)
     esperado: IFFALSE a GOTO L0
     obtenido: IFFALSE a GOTO L0
     -> OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion24.md
```

- [x] Caso base y límites probados por el harness sin *Crash*.
- [ ] No existen accesos fuera de límites ni *Memory Leaks* en todos los escenarios posibles.
- [x] La complejidad teórica coincide con la implementación para los casos probados.

### Reporte Crítico

*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*

> Al revisar el código de la IA, observé que no valida si el arreglo `lineas` es NULL. Si se recibe un arreglo inválido, la función podría intentar escribir en una dirección de memoria incorrecta. También noté que cuando `op` recibe un carácter diferente de `&` o `|`, la función lo interpreta como si fuera AND, porque utiliza `"IFFALSE"` como opción predeterminada. El harness no incluye pruebas para esos casos, por eso no puedo afirmar que estén corregidos. Como mejora, agregaría validaciones para el arreglo y para los operadores permitidos antes de generar las instrucciones.
>
> También comprobé que la IA utiliza `snprintf`, lo cual ayuda a evitar que una cadena sobrepase los 64 caracteres de cada línea. Sin embargo, no verifica si el texto quedó truncado. Las seis pruebas proporcionadas sí fueron superadas tanto por mi implementación manual como por la de IA, pero todavía sería conveniente probar entradas inválidas y límites de capacidad.

---

**Firma digital:** `josue6martin`
