/******************************/
/* File: tiny.flex            */
/* Lex specification for C-   */
/******************************/

%{
#include "globals.h"
#include "util.h"
#include "scan.h"
#include "util.c"

/* lexeme of identifier or reserved word */
char tokenString[MAXTOKENLEN+1];
%}

digit       [0-9]
number      {digit}+
letter      [a-zA-Z]
identifier  {letter}+
newline     \n
whitespace  [ \t]+

/* This tells flex to read only one input file */
%option noyywrap

%%


"if"            {return IF;}
                                    /* removed "then" because it does not exist in C- */
"else"          {return ELSE;}
"return"        {return RETURN;}    /* "end" to "return" */
"void"          {return VOID;}      /* added "void" */
"do"            {return DO;}        /* "do" instead of "repeat" */
"while"         {return WHILE;}     /* changed 'until' to 'while' */
"scanf"         {return SCAN;}      /* changed "read" to "scanf" */
"printf"        {return PRINT;}     /* changed "write" to "printf" */
"="             {return ASSIGN;}    /* changed ":=" to "="  */
"=="            {return EQ;}        /* changed "=" to "==" */
"<"             {return LT;}
"+"             {return PLUS;}
"-"             {return MINUS;}
"*"             {return TIMES;}
"/"             {return OVER;}
"("             {return LPAREN;}  /* added Left Parentheses */
")"             {return RPAREN;}  /* added Right Parentheses */
"["             {return LSBRACK;} /* added Left Square Brackets */
"]"             {return RSBRACK;} /* added Right Square Brackets */
"{"             {return LCBRACK;} /* added Left Curly Brackets */
"}"             {return RCBRACK;} /* added Right Curly Brackets */
";"             {return SEMI;}
{number}        {return NUM;}
{identifier}    {return ID;}
{newline}       {lineno++;}
{whitespace}    {/* skip whitespace */}
"/*"            { 
                    char c;
                    do {
                        c = input();
                        if (c == EOF) break;  
                        if (c == '\n') lineno++; 
                    } while (c != '*' || (c == '*' && input() != '/'));  // switched from '{}' comment to '/**/' comment 
                }


%%

TokenType getToken(void)
{ 
  static int firstTime = TRUE;
  TokenType currentToken;
  if (firstTime)
  { 
    firstTime = FALSE;
    lineno++;
    yyin = fopen("tiny.txt", "r+");
    yyout = fopen("result.txt", "w+");
    listing = yyout;
  }
  currentToken = yylex();
  strncpy(tokenString, yytext, MAXTOKENLEN);
  
  fprintf(listing, "\t%d: ", lineno);
  printToken(currentToken, tokenString);
  
  return currentToken;
}

int main()
{
    printf("Welcome to the C- flex scanner: ");
    while(getToken())
    {
        printf("A new token has been detected...\n");
    }
    return 1;
}
