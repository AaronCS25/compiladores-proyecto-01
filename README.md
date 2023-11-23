# Proyecto 01 - Compiladores CS(30816)

## Task 01:

Agregar a IMP0 la posibilidad de incluir comentarios de una sola línea en cualquier punto del
programa. Los comentarios deberán empezar con // y acabar con el fin de línea. Así, por ejemplo,
se podrá escribir código **IMP0** como el de abajo:

```c++
var int x, n; // variables globales
n = 10; // longitud
x = 0;
while x < n
	print(10** x) // imprimir potencia de 10
	x = x+1;
endwhile
```

**Reporte:**

EL cambio realizado para poder aceptar comentarios en el lenguaje IMP se realizó en el método nextToken() de la clase Scanner. Este método es el encargado de identificar los tokens y retornarlos, por lo que es en este punto en el que se deberían ignorar los comentarios. La estrategia a utilizar fue similar a cómo se ignoran los espacios en blanco:

- Reconocemos los símbolos que indican el inicio de comentarios.
- Seguimos leyendo caracteres hasta encontrar un salto de linea.
- Se hace rollback() o nextChar() según sea conveniente para identificar los tokens después de los comentarios.

```cpp
Token* Scanner::nextToken() {
  Token* token;
  char c;
  // consume whitespaces
  c = nextChar();
  while (c == ' ' || c == '\t'  || c == '\n') c = nextChar();
  if (c == '\0') return new Token(Token::END);
  //--------------- CODIGO AGREGADO ---------------------//
  if (c == '/') // MANEJO DE COMENTARIOS
  {
    // ¿Es comentario?
    c = nextChar();
    if (c == '/') 
    {
      // Si es comentario -> ignorar todos los caracteres hasta el primer salto de línea ('\n').
      while (c != '\n') c = nextChar();
      c = nextChar();
    }
    else {  rollBack(); } // Si no es comentario hacer rollback().
    rollBack();
    c = nextChar(); // Volver a setear c
  }
   
  //------------------------- FIN ------------------------//
  startLexema();
    .
    .
    .
```

