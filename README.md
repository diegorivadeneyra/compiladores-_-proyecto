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
