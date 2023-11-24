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

EL cambio realizado para poder aceptar comentarios en el lenguaje IMP se realizó en el método nextToken() de la clase Scanner. Este método es el encargado de identificar los tokens y retornarlos, por lo que es en este punto en el que se deberían ignorar los comentarios. La parte donde se realizó el cambio fue específicamente en la sección de las condiciones donde se reconoce el símbolo de alguna operación. La estrategia a utilizar fue similar a cómo se ignoran los espacios en blanco y a cómo se diferencia el operador de asignación con el de comparación:

- Leemos un caracter igual a '/'.
- Si el siguiente caracter también es igual a '/', significa que hemos encontrado un comentario. En ese caso seguimos leyendo caracteres hasta encontrar un salto de linea. Finalmente, se retornará el siguiente token al llamar la función nextToken() recursivamente.
- Caso contrario, hacemos rollback para ubicarnos en el primer caracter '/' y retornamos el token de división.

```cpp
    case '/':
      c = nextChar();
      if (c == '/')
      {
        while (c != '\n') c = nextChar();
        token = this->nextToken();
      }
      else { rollBack(); token = new Token(Token::DIV); }
      break;
```

## Task 02:


## Task 03: