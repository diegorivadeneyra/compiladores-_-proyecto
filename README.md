# compiladores-_-proyecto

### Integrantes

- Rivadeneyra Diego
- Lizarraga Sebastian
- Mora Angel


### 1-comentarios

Primero creamos el nuevo TOKEN para los comentarios "COMENT" tanto en el parcer.h como en el parcer.cpp
```
    const char *Token::token_names[38] = {
    "LPAREN", "RPAREN",    "PLUS",  "MINUS",  "MULT",    "DIV",
    "EXP",    "LT",        "LTEQ",  "EQ",     "NUM",     "ID",
    "PRINT",  "SEMICOLON", "COMMA", "ASSIGN", "CONDEXP", "IF",
    "THEN",   "ELSE",      "ENDIF", "WHILE",  "DO",      "ENDWHILE",
    "ERR",    "END",       "VAR",   "NOT",    "TRUE",    "FALSE",
    "AND",    "OR",        "FOR",   "COLON",  "ENDFOR",  "COMENT", "CONTINUE", "BREAK",
    };

```

Modificamos el scanner para la descomposicion de nuestro ejemplo y que el guardado del Token correspondiente en el stack. Para ello, solo modificamos el caso de la division ya que nuestro valor para sver que es un comentario es "//", por tanto, si luego ded una barra existe otra significara que es unu comentario y se comera todo lo quue siga hasta quie encuentre el final o un salto de linea.

```
case '/':
      c = nextChar();
      if (c == '/') {
        c = nextChar();
        while (c != '\0' && c != '\n') {
          c = nextChar();
        }
        rollBack();
        token = new Token(Token::COMENT, getLexema());
      } else {
        rollBack();
        token = new Token(Token::DIV);
      }
      break;
```
Ademas, se modifica en el parser para que cuando recepcione los TOKEN de comentarios simplemente los ignore o los pase de largo.
Este cambio se realiza en el StatementList asi se verifica el final de cada linea si existe un ";" verifica si existe un comentario luego para obviarlo. Asimismo, si no encuentra un ";" significa que es la ultima instruccion y tambien revisa la existencia de algun comentario para obivviarlo. 

```
StatementList *Parser::parseStatementList() {
  StatementList *p = new StatementList();
  p->add(parseStatement());
  while (match(Token::SEMICOLON)) {
    while (match(Token::COMENT)) {
      // do nothing
    }
    p->add(parseStatement());
  }
  while (match(Token::COMENT)) {
    // do nothing
  }
  return p;
}

```



### 2-Generacion de codigo 

### 3-Sentencia do-while
Para incorporar la instrucción `do while`, se crearon las clases `DoWhileStatement` y se añadió la siguiente lectura al analizador sintáctico:

```cpp
} else if (match(Token::DO)) {
    tb = parseBody();
    if (!match(Token::WHILE)) {
      parserError("Se esperaba un while");
    }
    e = parseExp();
    s = new DoWhileStatement(e, tb);
}
```
De esta manera, se logra la lectura del bloque (body) y la expresión. Dado que comparten las mismas palabras clave que el while, no fue necesario realizar más cambios en el analizador sintáctico.

En el caso del typechecker, la lógica es similar a la del while, ya que los tipos que deben contener son los mismos:
```cpp
void ImpTypeChecker::visit(DoWhileStatement *s) {
  s->body->accept(this);
  if (!s->cond->accept(this).match(booltype)) {
    cout << "La condición en DoWhileStm debe ser de tipo: " << booltype << endl;
    exit(0);
  }
  return;
}
```
Para el interpreter, se realizó un cambio en comparación con el while. El DoWhile ejecuta primero el bloque (body) antes de verificar la condición. Por lo tanto, el interpreter resultante debe seguir esa estructura:
```cpp
int ImpInterpreter::visit(DoWhileStatement *s) {
  s->body->accept(this);
  while (s->cond->accept(this)) {
    if (this->breakFlag == true) {
      breakFlag = false;
      break;
    }
    s->body->accept(this);
  }
  return 0;
}
```
Finalmente, para el codegen, se deben generar dos etiquetas: una para dirigirse al inicio del bucle y la otra para salir del bucle. Dado que la estructura implica ejecutar primero el bloque y luego verificar la condición, el código generado es el siguiente:
```cpp
int ImpCodeGen::visit(DoWhileStatement* s) {
  string l1 = next_label();
  string l2 = next_label();
  this->breakLabel = l2; 
  this->continueLabel = l1; 

  codegen(l1, "skip");
  s->body->accept(this);
  s->cond->accept(this);
  codegen(nolabel, "jmpz", l2);
  codegen(nolabel, "goto", l1);
  codegen(l2, "skip");
  this->breakLabel = ""; 
  this->continueLabel = ""; 
  return 0;
}
```
En este código, se ejecuta primero el body y luego se evalúa la expresión. Si la expresión es falsa, se ejecuta el jmpz, permitiendo la salida del bucle.


### 4-Sentencia break y continue
### Agregar las instrucciones "Break" y "Continue"

Para incorporar las instrucciones "Break" y "Continue", se introdujeron los tokens correspondientes en el código:

```cpp
"CONTINUE", "BREAK"
```

Además, se crearon las clases `ContinueStatement` y `BreakStatement`, las cuales no contienen body ni expresión.

Dentro del analizador léxico (scanner), se reservaron las palabras "continue" y "break" para asignarles los tokens correspondientes. En el analizador sintáctico, se agregaron las siguientes líneas al método `parseStatement`:

```cpp
} else if(match(Token::CONTINUE)){
    s = new ContinueStatement();
  } else if(match(Token::BREAK)){
    s = new BreakStatement();
  }
```

En el `typechecker`, se añadieron los correspondientes visitantes para estas nuevas clases, aunque al no tener variables que requieran verificación de tipo, los visitantes quedaron vacíos.

En el `interpreter`, se introdujeron las variables `continueFlag` y `breakFlag` en la clase `ImpInterpreter`, las cuales sirven para la conexión entre `ContinueStatement`/`BreakStatement` y su bucle padre. Los visitantes de estos nuevos Statements dentro del interpreter modifican estas variables:

```cpp
int ImpInterpreter::visit(ContinueStatement *s) {
  this->continueFlag = true;
  return 0;
}

int ImpInterpreter::visit(BreakStatement *s) {
  this->breakFlag = true;
  return 0;
}
```

Esto llevó a modificar los visitantes de `WhileStatement` y `DoWhileStatement` para incorporar una condición correspondiente al `breakFlag`, permitiendo que, si es verdadero, se restablezca a falso y se rompa el bucle:

```cpp
int ImpInterpreter::visit(WhileStatement *s) {
  while (s->cond->accept(this)) {
    if (this->breakFlag == true) {
      breakFlag = false;
      break;
    }
    s->body->accept(this);
  }
  return 0;
}
```

En el caso de `StatementList`, este Statement que lee los contenidos del body también tuvo que modificarse. En el caso de `break` y `continue`, todo el código que siga debe omitirse. Para esto, el `continueFlag` toma el valor de falso si su condición se cumple, permitiendo que el bucle while continúe y solo se salten las instrucciones en caso de que el flag sea true.

```cpp
int ImpInterpreter::visit(StatementList *s) {
  list<Stm *>::iterator it;
  for (it = s->slist.begin(); it != s->slist.end(); ++it) {
    if (this->breakFlag == true || this->continueFlag == true) {
      this->continueFlag = false; 
      break;
    }
    (*it)->accept(this);
  }
  return 0;
}
```

Finalmente, en el `codegen`, se agregaron las variables `continueLabel` y `breakLabel` en `ImpCodeGen`. Estas variables se utilizan para conectar las etiquetas de los bucles con `ContinueStatement`/`BreakStatement`. Para `ContinueStatement`, se conecta al label que inicia nuevamente el bucle, y para `BreakStatement`, se conecta al label que apunta al final del bucle. Los Statement tendrán el siguiente codegen:

```cpp
int ImpCodeGen::visit(ContinueStatement* s) {
  codegen(nolabel, "goto", this->continueLabel);
  return 0;
}

int ImpCodeGen::visit(BreakStatement* s) {
  codegen(nolabel, "goto", this->breakLabel);
  return 0;
}
```

Es importante mencionar que esta implementación de `break` y `continue` tiene limitaciones. La principal es que no se puede generar un bucle dentro de otro, ya que se sobrescribirían las variables `continueLabel` y `breakLabel`, perdiendo los puntos de referencia para definir el codegen. Otra limitación es que, para el interpreter, las instrucciones `continue` y `break` deben estar dentro de un `IfStatement` que solo contenga esa instrucción. Esta decisión se tomó para asegurar que al finalizar la lectura del `IfStatement`, las variables `continueFlag` y `breakFlag` se mantengan en true, permitiendo volver al bucle `while` y afectarlo. Estas limitaciones surgen al intentar conectar los nuevos Statements con sus respectivos bucles. Una solución potencial sería generar un tipo de entorno que almacene cada `continue` y `break` en su bucle padre, guardando los labels de inicio y salida del bucle para evitar sobrescribirlos y permitir una conexión más segura y personalizada entre Statements.
