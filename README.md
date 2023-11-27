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

### 4-Sentencia break y continue
