# Proyecto 01 - Compiladores CS(30816)

Integrantes:
- Aaron Camacho
- Mauricio Álvarez
- Pedro Mori

## Task 01:

Agregar a IMP0 la posibilidad de incluir comentarios de una sola línea en cualquier punto del
programa. Los comentarios deberán empezar con // y acabar con el fin de línea. Así, por ejemplo,
se podrá escribir código **IMP0** como el de abajo:

```cpp
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

La tarea 3 pedía la implementación de constantes booleanas,los operadores and/or y una versión 
de foor lops. El código que acompaña este proyecto tiene estas implementaciones. Además 
contiene implementaciones vacías del Visitor que implementa codegen (para permitir que todo 
compile). 
Implementar la generación de código objeto para constantes booleanas, los operadores and/or 
y la versión de for loops considerada por el intérprete: solo considera for-loops con valores 
ascendentes. Por ejemplo:

```cpp
for x : 5, 10 do Body endfor   // ejecuta Body 6 veces con x :  5,6,...,10 pero 
for x : 10, 5 do Body endfor  // no ejecuta Body ni una sola vez. 
```

**Reporte**

Generación de código para constantes booleanas:

```cpp
int ImpCodeGen::visit(BoolConstExp* e) {
  int value = e->b ? 1 : 0;
  codegen(nolabel, "push", value);
  return 0;
}
```

- Obtenemos el valor de la constante booleana (true o false).
- Generamos código para la instrucción de push 1 (true) o push 0 (false) en la pila.

Generación de código para operadores AND/OR:

```cpp
int ImpCodeGen::visit(BinaryExp* e) {
  e->left->accept(this);
  e->right->accept(this);
  string op = "";
  switch(e->op) {
  case PLUS: op =  "add"; break;
  case MINUS: op = "sub"; break;
  case MULT:  op = "mul"; break;
  case DIV:  op = "div"; break;
  case LT:  op = "lt"; break;
  case LTEQ: op = "le"; break;
  case EQ:  op = "eq"; break;
  case AND: op = "and"; break; // NUEVO
  case OR: op = "or"; break; // NUEVO
  default: cout << "binop " << Exp::binopToString(e->op) << " not implemented" << endl;
  }
  codegen(nolabel, op);
  return 0;
}
```

- Se agregaron los casos para AND y OR en el visit de BinaryExp, ya que es ahí donde se genera código para los operadores bianarios.

Generación de código para FOR statement:

```cpp
int ImpCodeGen::visit(ForStatement* s) {
  string l1 = next_label();
  string l2 = next_label();

  s->e1->accept(this);
  codegen(nolabel, "store", direcciones.lookup(s->id));
  codegen(l1,"skip");
  s->e2->accept(this);
  codegen(nolabel,"load",direcciones.lookup(s->id));
  codegen(nolabel,"lt");
  codegen(nolabel,"jmpz",l2);
  s->body->accept(this);
  codegen(nolabel,"load",direcciones.lookup(s->id));
  codegen(nolabel, "push", 1);
  codegen(nolabel, "add");
  codegen(nolabel, "store", direcciones.lookup(s->id));
  codegen(nolabel,"goto",l1);
  codegen(l2,"skip");
  return 0;
}
```

Recordemos que un FOR loop para el lenguaje trabajado en el curso tiene la siguiente estructura:

```cpp
for x : 5, 10 do Body endfor
```

En este sentido, los pasos a seguir para su generación de código serían los siguientes:

- Generar código para la expresión antes de la coma y después del token COLON.
- Guardamos ese valor que se encontrará en el top de la pila en la dirección de la variable a iterar del loop.
- Creamos un primer label que nos permitirá relizar el loop.
- Generamos código para la segunda expresión después de la coma y antes del token DO.
- Traemos al top de la pila el valor de la variable a iterar.
- Comparamos si es menor a la segunda expresión.
- Si es falso, rompemos el loop y vamos a un segundo label auxiliar.
- Generamos código para el body.
- Nuevamente traemos al top de la pila el valor de la variable a iterar, le sumamos 1, lo guardamos en su dirección correspondiente y regresamos al inicio del label.


## Task 03:

Implementar la interpretación estándar de do-while. Por ejemplo: 
  
```cpp
  x= 0; do x = x+ 1; print(x) while x < 5  // imprime 1,2,3,4,5 
```  
Nótese que el do-while no necesita un marcador como enddo. Para delimitar el fin de la 
sentencia. ¿Se puede implementar el parser? Pero, puede agregarse algo asi si lo desean.

**Reporte**

Se creó una clase DoWhileStatement heredada por la clase Statement con la siguiente estructura:

```cpp
class DoWhileStatement : public Stm {
public:
  Exp* cond;
  Body *body;
  DoWhileStatement(Body* b, Exp* c);
  int accept(ImpVisitor* v);
  void accept(TypeVisitor* v);
  ~DoWhileStatement();
};
```

Vemos que en realida tiene la misma estructura que el WhileStatement, lo que la diferenciará es en el código del printer, intérprete, typecheck y codegen.

Para el Parser, se agregó un caso más al método de parseStatement():

```cpp
  else if (match(Token::DO)) {
    tb = parseBody();
    if (!match(Token::WHILE)) parserError("Esperaba WHILE en do-while");
    e = parseExp();
    s = new DoWhileStatement(tb, e);        
  }
```

Para el printer, se agregó un método visit que reciba como parámetro un objeto de tipo DoWhileStatement:

```cpp
int ImpPrinter::visit(DoWhileStatement* s) {
  cout << "do " << endl;
  s->body->accept(this);
  cout << "while (";
  s->cond->accept(this);
  cout << ")";
  return 0;
}
```
Como sabemos, la diferencia del do-while con el while loop es que el do-while ejecuta al menos una vez las instrucciones del body y luego evalúa la condición. Esto explica el orden en el que se encuentran las implementaciones de los métodos visit() en el intérprete y el typechecker.


Intérprete:

```cpp
int ImpInterpreter::visit(DoWhileStatement* s) {
  s->body->accept(this);
  while (s->cond->accept(this)) {
    s->body->accept(this);
  }
  return 0;
}
```

Typechecker:

```cpp
void ImpTypeChecker::visit(DoWhileStatement* s) {
  s->body->accept(this);
  if (!s->cond->accept(this).match(booltype)) {
    cout << "Condicional en DoWhileStatement debe de ser: " << booltype << endl;
    exit(0);
  }  
  return;
}
```

Por último, para la implementación del codegen, se utilizó como base el codegen del WhileStatement, solo que se varío el orden para cumplir con el comportamiento esperado.

```cpp
int ImpCodeGen::visit(DoWhileStatement* s) {
  string l1 = next_label();
  string l2 = next_label();

  codegen(l1,"skip");
  s->body->accept(this); //Generamos código para el body
  s->cond->accept(this); //Generamos código para la codición
  codegen(nolabel,"jmpz",l2); //Salimos del loop si no se cumple la condición
  codegen(nolabel,"goto",l1); //Seguimos la iteración
  codegen(l2,"skip");
  return 0;
}
```

## Task 04:

Implementar las interpretaciones estándar de break y continue, las sentencias que permiten 
salir y terminar un loop o saltar a la condición de control del loop. Notar que el salto asociado a un 
continue es distinto para un while-do que para un do-while (al comienzo o al final). 
 
No es necesario implementar la parte asociada al interprete.

**Reporte**

Se crearon dos clases BreakStatement y ContinueStatement heredada por la clase Statement con la siguiente estructura:

```cpp
class BreakStatement : public Stm {
public:
  BreakStatement();
  int accept(ImpVisitor* v);
  void accept(TypeVisitor* v);
  ~BreakStatement();
};
```

```cpp
class ContinueStatement : public Stm {
public:
  ContinueStatement();
  int accept(ImpVisitor* v);
  void accept(TypeVisitor* v);
  ~ContinueStatement();
};
```


El primer cambio que se realizó fue agregar a BREAK y CONTINUE como tipos de Token a reconocer por el Scanner y también en el hash table de palabras reservadas.

```cpp
  enum Type { ..., BREAK, CONTINUE };

  const char* Token::token_names[37] = {..., "BREAK", "CONTINUE" };

  reserved["break"] = Token::BREAK;
  reserved["continue"] = Token::CONTINUE;
```

Luego, se realizaron cambios a la gramática del lenguaje IMP0, específicamente en el Stm:

```ebnf
Stm ::= ... | 'break' | 'continue' 
```

Por ello, a nivel de código del Parser se modificó el método parseStatement(), donde se agregaron los casos para reconocer BREAK o CONTINUE.

```cpp
else if (match(Token::BREAK)) {
  s = new BreakStatement();
}
else if (match(Token::CONTINUE)) {
  s = new ContinueStatement();
}
```

La implementación de los métodos visit() en el Printer es relativamente sencillo.

```cpp
int ImpPrinter::visit(BreakStatement* s) {
  cout << "break" << endl;
  return 0;
}

int ImpPrinter::visit(ContinueStatement* s) {
  cout << "continue" << endl;
  return 0;
}
```

Para la implementación del TypeChecker, debido a que tanto BREAK como CONTINUE son instrucciones sin argumentos, la única verificación semántica que se debe hacer según lo solicitado es que sean ejecutados dentro de un loop (FOR, WHILE, DO-WHILE). Para ello, se agregó un flag booleano en la clase ImpTypeChecker, el cual se será verdadero cada vez que entremos a un loop y falso cuando termine de ejecutarse.

```cpp
class ImpTypeChecker : public TypeVisitor
public:
  ImpTypeChecker();
private:
  Environment<ImpType> env;
  ImpType booltype;
  ImpType inttype;
  bool loopFlag = false; // Flag Agregado
```

```cpp
void ImpTypeChecker::visit(WhileStatement* s) {
  this->loopFlag = true; // Agregado
  if (!s->cond->accept(this).match(booltype)) {
    cout << "Condicional en WhileStm debe de ser: " << booltype << endl;
    exit(0);
  }  
  s->body->accept(this);
  this->loopFlag = false; // Agregado
 return;
}

void ImpTypeChecker::visit(ForStatement* s) {
  this->loopFlag = true; // Agregado
  ImpType t1 = s->e1->accept(this);
  ImpType t2 = s->e2->accept(this);
  if (!t1.match(inttype) || !t2.match(inttype)) {
    cout << "Tipos de rangos en for deben de ser: " << inttype << endl;
    exit(0);
  }
  env.add_level();
  env.add_var(s->id,inttype);
  s->body->accept(this);
  env.remove_level();
  this->loopFlag = false; // Agregado
  return;
}

void ImpTypeChecker::visit(DoWhileStatement* s) {
  this->loopFlag = true; // Agregado
  s->body->accept(this);
  if (!s->cond->accept(this).match(booltype)) {
    cout << "Condicional en DoWhileStatement debe de ser: " << booltype << endl;
    exit(0);
  }
  this->loopFlag = false; // Agregado   
  return;
}
```

Finalmente, el typecheck de BREAK y CONTINUE se basará en este flag. Si alguna de las instrucciones es puesta fuera de un loop, se imprimirá el error y terminará la ejecución del programa.

```cpp
void ImpTypeChecker::visit(BreakStatement* s) {
  if (!this->loopFlag) {
    cout << "BreakStatement fuera de un loop" << endl;
    exit(0);
  }  
  return;
}

void ImpTypeChecker::visit(ContinueStatement* s) {
  if (!this->loopFlag) {
    cout << "ContinueStatement fuera de un loop" << endl;
    exit(0);
  }  
  return;
}
```

Para la implementación de la generación de código para estas dos nuevas instrucciones, agregamos dos nuevas variables auxiliares en la clase ImpCodeGen donde se almacenarán los label inicial y final al cual cada instruccion debe saltar del loop actual.

```cpp
class ImpCodeGen : public ImpVisitor 
private:
  std::ostringstream code;
  string nolabel;
  int current_label;
  Environment<int> direcciones;
  int siguiente_direccion, mem_locals;
  string initLabel, endLabel; // Agregado
  void codegen(string label, string instr);
  void codegen(string label, string instr, int arg);
  void codegen(string label, string instr, string jmplabel);
  string next_label();
```

Ahora, cada instrucción saltará al label correspondiente. En el caso de BREAK, siempre debería saltar al label designado para salir de un loop. Para CONTINUE, aquí sí hay diferencias según el tipo de loop en el que nos encontremos:

- WHILE: Debe volver al label inicial, ya que es aquí donde se evalúa la condición sin problemas.
- DO-WHILE: Debe existir un label auxiliar para marcar donde inicia la condición y CONTINUE debería saltar aquí, ya que el label inicial para este loop empieza evaluando el body.
- FOR: Debe existir un label auxiliar para marcar donde inicia el aumento de valor de la variable a iterar y CONTINUE debería saltar aquí, ya que es luego de esto que se evalúa la condición.


Modificaciones para WHILE LOOP

```cpp
int ImpCodeGen::visit(WhileStatement* s) {
  string l1 = next_label();
  string l2 = next_label();

  this->initLabel = l1; // Agregado
  this->endLabel = l2; // Agregado

  codegen(l1,"skip");
  s->cond->accept(this);
  codegen(nolabel,"jmpz",l2);
  s->body->accept(this);
  codegen(nolabel,"goto",l1);
  codegen(l2,"skip");

  this->initLabel = ""; // Agregado
  this->endLabel = ""; // Agregado
  return 0;
}
```

Modificaciones para DO-WHILE LOOP

```cpp
int ImpCodeGen::visit(DoWhileStatement* s) {
  string l1 = next_label();
  string contidionLabel = next_label(); // Agregado
  string l2 = next_label();

  this->initLabel = contidionLabel; // Agregado - Diferencia con While Loop y For Loop
  this->endLabel = l2; // Agregado

  codegen(l1,"skip");
  s->body->accept(this);
  codegen(contidionLabel,"skip"); // Agregado
  s->cond->accept(this);
  codegen(nolabel,"jmpz",l2);
  codegen(nolabel,"goto",l1);
  codegen(l2,"skip");

  this->initLabel = ""; // Agregado
  this->endLabel = ""; // Agregado
  return 0;
}
```

Modificaciones para FOR LOOP

```cpp
int ImpCodeGen::visit(ForStatement* s) {
  string l1 = next_label();
  string updateLabel = next_label(); // Agregado
  string l2 = next_label();

  this->initLabel = updateLabel; // Agregado - Diferencia con While Loop y Do-While loop
  this->endLabel = l2;

  s->e1->accept(this);
  codegen(nolabel, "store", direcciones.lookup(s->id));
  codegen(l1,"skip");
  s->e2->accept(this);
  codegen(nolabel,"load",direcciones.lookup(s->id));
  codegen(nolabel,"lt");
  codegen(nolabel,"jmpz",l2);
  s->body->accept(this);
  codegen(updateLabel,"skip"); // Agregado
  codegen(nolabel,"load",direcciones.lookup(s->id));
  codegen(nolabel, "push", 1);
  codegen(nolabel, "add");
  codegen(nolabel, "store", direcciones.lookup(s->id));
  codegen(nolabel,"goto",l1);
  codegen(l2,"skip");
  
  this->initLabel = "";  //Agregado
  this->endLabel = ""; // Agregado
  return 0;
}
```

Finalmente, los métodos visit() para la generación de código de BREAK y CONTINUE serían de la siguiente forma:

```cpp
int ImpCodeGen::visit(BreakStatement* s) {
  codegen(nolabel,"goto",this->endLabel);
  return 0;
}

int ImpCodeGen::visit(ContinueStatement* s) {
  codegen(nolabel,"goto",this->initLabel);
  return 0;
}
```