# Taller – Ejercicios capitulo 2 flex

## Concordancia con Tabla Hash Dinámica

En este taller se trabajaron los ejercicios del capítulo 2 implementándolos en **Lex** sobre Kali Linux.
El objetivo fue modificar el programa de concordancia para:

* Analizar correctamente los patrones de líneas.
* No distinguir entre mayúsculas y minúsculas.
* Evitar que la tabla hash se bloquee cuando se llena.

A continuación se explica paso a paso lo realizado y las decisiones tomadas durante el desarrollo.

---

# 1️ Instalación y Preparación en Kali Linux

Primero se verificó si `lex` estaba instalado:

```bash
lex --version
```

Como Kali utiliza realmente `flex` (compatible con lex), se instaló lo necesario:

```bash
sudo apt update
sudo apt install flex bison gcc build-essential -y
```

Luego se creó una carpeta para el taller:

```bash
mkdir taller_lex
cd taller_lex
```

Después se creó el archivo:

```bash
nano concordancia.l
```

En ese archivo se escribió el programa correspondiente.

---

# 2️ Punto 1 – ¿Por qué no usar `^.*\n`?

El ejemplo original compara carácter por carácter. Surge la pregunta: ¿por qué no usar simplemente `^.*\n` para comparar línea por línea?

La razón es que en Lex:

* El punto `.` **no incluye el salto de línea (`\n`)**.
* `^` solo funciona realmente al inicio de línea.
* `.*` puede fallar si la última línea no termina con salto de línea.
* Puede volverse demasiado general o no comportarse como se espera.

Por eso `^.*\n` no es un patrón confiable en Lex.

## Patrón correcto utilizado

En su lugar se propone:

```lex
^[^\n]*\n
```

Este patrón significa:

* `^` → inicio de línea
* `[^\n]*` → cualquier carácter excepto salto de línea
* `\n` → fin de línea

Si se desea capturar fragmentos más grandes (por ejemplo varias líneas consecutivas), se puede utilizar:

```lex
([^\n]*\n)+
```

Este patrón sí funciona correctamente dentro del funcionamiento interno de Lex.

---

# 3️ Punto 2 – No distinguir mayúsculas y minúsculas

El programa original trataba palabras como:

```
Casa
casa
CASA
```

como si fueran diferentes.

Se modificó el programa para que las gestione como la misma palabra, sin crear copias adicionales innecesarias.

## 🔧 Cambios realizados

### En la función `symhash()`:

Se convirtió cada carácter a minúscula antes de calcular el hash:

```c
hash = hash * 9 ^ tolower(c);
```

### En la comparación de palabras:

En lugar de usar `strcmp`, se utilizó:

```c
strcasecmp(sp->name, sym)
```

Esto permite comparar palabras ignorando las diferencias entre mayúsculas y minúsculas.

Con esta modificación el programa cuenta correctamente las palabras sin importar cómo estén escritas.

---

# 4️ Punto 3 – Tabla Hash Dinámica

El programa original utilizaba una tabla de tamaño fijo y se detenía si esta se llenaba.

Existen dos técnicas estándar para resolver este problema:

* Encadenamiento
* Rehashing

En este taller se implementó **encadenamiento**.

##  Razón de la elección

Se eligió encadenamiento porque:

* Es más sencillo de implementar.
* No requiere mover elementos existentes.
* No afecta la estructura cuando la tabla crece.
* Es más práctico para el caso de referencia cruzada.

El rehashing complica más el proceso porque obliga a crear una nueva tabla más grande y trasladar todos los elementos, recalculando sus posiciones.

##  Implementación realizada

Cada posición de la tabla se convirtió en una lista enlazada:

```c
struct symbol {
    char *name;
    int count;
    struct symbol *next;
};
```

Cuando ocurre una colisión:

```c
sp->next = symtab[hash];
symtab[hash] = sp;
```

De esta manera la tabla no se bloquea, ya que puede crecer dinámicamente utilizando `malloc()`.

---

# 5️ Compilación

Primero se generó el archivo en C:

```bash
flex concordancia.l
```

Luego se compiló:

```bash
gcc lex.yy.c -o concordancia -lfl
```

Finalmente se ejecutó:

```bash
./concordancia
```

También se probó con un archivo de entrada:

```bash
./concordancia < texto.txt
```

---

# 6️ Conclusión

En este taller se logró:

* Comprender por qué `^.*\n` no funciona correctamente en Lex.
* Proponer un patrón adecuado para líneas completas.
* Modificar el hash para ignorar mayúsculas y minúsculas.
* Implementar una tabla hash dinámica utilizando encadenamiento.
* Compilar y ejecutar correctamente el programa en Kali Linux.

Este ejercicio permitió reforzar la comprensión del funcionamiento interno de Lex y el manejo de memoria dinámica en C dentro de un analizador léxico.

---
