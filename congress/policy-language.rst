===============
Policy language
===============

The policy language for Congress is Datalog, which is basically SQL but with
a syntax that is closer to traditional programming languages. This declarative
language was chosen because its semantics are well-known to a broad range of
DevOps, yet its syntax is more terse making it better suited for expressing
real-world policies.

The core grammar is given below::

<policy> ::= <rule>*
<rule> ::= <atom> COLONMINUS <literal> (COMMA <literal>)*
<literal> ::= <atom>
<literal> ::= NOT <atom>
<atom> ::= TABLENAME LPAREN <term> (COMMA <term>)* RPAREN
<term> ::= INTEGER | FLOAT | STRING | VARIABLE
