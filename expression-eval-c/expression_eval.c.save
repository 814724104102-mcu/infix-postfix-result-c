// expression_eval.c
// Compile: gcc -o expression_eval expression_eval.c -lm
// Example: ./expression_eval "3 + 4 * (2 - 1)"

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <ctype.h>
#include <math.h>

typedef enum { TOK_NUMBER, TOK_OP, TOK_LPAREN, TOK_RPAREN } TokenType;

typedef struct {
    TokenType type;
    char *text;
} Token;

typedef struct {
    Token *data;
    int size;
    int cap;
} TokenArray;

void ta_init(TokenArray *ta) { ta->data = NULL; ta->size = 0; ta->cap = 0; }
void ta_free(TokenArray *ta) {
    for (int i = 0; i < ta->size; ++i) free(ta->data[i].text);
    free(ta->data);
    ta->data = NULL; ta->size = ta->cap = 0;
}
void ta_push(TokenArray *ta, Token t) {
    if (ta->size == ta->cap) {
        ta->cap = ta->cap ? ta->cap * 2 : 8;
        ta->data = realloc(ta->data, sizeof(Token) * ta->cap);
    }
    ta->data[ta->size++] = t;
}

int is_operator_char(char c) {
    return c=='+' || c=='-' || c=='*' || c=='/' || c=='^';
}
int precedence(char op) {
    if (op == '+' || op == '-') return 1;
    if (op == '*' || op == '/') return 2;
    if (op == '^') return 3;
    return 0;
}
int is_right_associative(char op) { return op == '^'; }

Token make_token(TokenType type, const char *s) {
    Token t;
    t.type = type;
    t.text = strdup(s);
    return t;
}

TokenArray infix_to_postfix(const char *expr) {
    TokenArray output; ta_init(&output);
    TokenArray opstack; ta_init(&opstack);

    int i = 0;
    int len = strlen(expr);
    int expect_unary = 1;

    while (i < len) {
        if (isspace((unsigned char)expr[i])) { i++; continue; }

        if (isdigit((unsigned char)expr[i]) || expr[i] == '.') {
            int j = i;
            while (j < len && (isdigit((unsigned char)expr[j]) || expr[j] == '.')) j++;
            int n = j - i;
            char *num = malloc(n + 1);
            strncpy(num, expr + i, n);
            num[n] = '\0';
            ta_push(&output, make_token(TOK_NUMBER, num));
            free(num);
            i = j;
            expect_unary = 0;
            continue;
        }

        char c = expr[i];

        if (c == '(') {
            ta_push(&opstack, make_token(TOK_LPAREN, "("));
            i++;
            expect_unary = 1;
            continue;
        }

        if (c == ')') {
            int found = 0;
            while (opstack.size) {
                Token top = opstack.data[opstack.size - 1];
                if (top.type == TOK_LPAREN) { found = 1; free(top.text); opstack.size--; break; }
                ta_push(&output, make_token(TOK_OP, top.text));
                free(top.text);
                opstack.size--;
            }
            if (!found) fprintf(stderr, "Mismatched parentheses\n");
            i++;
            expect_unary = 0;
            continue;
        }

        if (is_operator_char(c)) {
            if (expect_unary && (c == '-' || c == '+')) {
                if (c == '-') {
                    ta_push(&output, make_token(TOK_NUMBER, "0"));
                } else {
                    i++; expect_unary = 1; continue;
                }
            }
            while (opstack.size) {
                Token top = opstack.data[opstack.size - 1];
                if (top.type != TOK_OP) break;
                char topc = top.text[0];
                char curc = c;
                int ptop = precedence(topc);
                int pcur = precedence(curc);
                if (( !is_right_associative(curc) && ptop >= pcur ) ||
                    ( is_right_associative(curc) && ptop > pcur)) {
                    ta_push(&output, make_token(TOK_OP, top.text));
                    free(top.text);
                    opstack.size--;
                } else break;
            }
            char s[2] = {c, '\0'};
            ta_push(&opstack, make_token(TOK_OP, s));
            i++;
            expect_unary = 1;
            continue;
        }

        fprintf(stderr, "Unknown character: '%c'\n", c);
        i++;
    }

    while (opstack.size) {
        Token top = opstack.data[opstack.size - 1];
        if (top.type == TOK_LPAREN || top.type == TOK_RPAREN) {
            fprintf(stderr, "Mismatched parentheses\n");
            free(top.text);
            opstack.size--;
            continue;
        }
        ta_push(&output, make_token(TOK_OP, top.text));
        free(top.text);
        opstack.size--;
    }

    ta_free(&opstack);
    return output;
}

typedef struct {
    double *data;
    int size;
    int cap;
} DStack;
void ds_init(DStack *s) { s->data = NULL; s->size = 0; s->cap = 0; }
void ds_free(DStack *s) { free(s->data); s->data = NULL; s->size = s->cap = 0; }
void ds_push(DStack *s, double v) {
    if (s->size == s->cap) { s->cap = s->cap ? s->cap*2 : 8; s->data = realloc(s->data, s->cap * sizeof(double)); }
    s->data[s->size++] = v;
}
double ds_pop(DStack *s) { if (s->size == 0) { fprintf(stderr, "Stack underflow\n"); return 0; } return s->data[--s->size]; }

int evaluate_postfix(const TokenArray *post, double *result) {
    DStack st; ds_init(&st);
    for (int i = 0; i < post->size; ++i) {
        Token t = post->data[i];
        if (t.type == TOK_NUMBER) {
            double v = strtod(t.text, NULL);
            ds_push(&st, v);
        } else if (t.type == TOK_OP) {
            if (st.size < 2) { fprintf(stderr, "Not enough operands for %s\n", t.text); ds_free(&st); return 0; }
            double b = ds_pop(&st);
            double a = ds_pop(&st);
            char op = t.text[0];
            double res = 0;
            switch (op) {
                case '+': res = a + b; break;
                case '-': res = a - b; break;
                case '*': res = a * b; break;
                case '/': if (b == 0) { fprintf(stderr, "Division by zero\n"); ds_free(&st); return 0; } res = a / b; break;
                case '^': res = pow(a, b); break;
                default: fprintf(stderr, "Unknown operator %c\n", op); ds_free(&st); return 0;
            }
            ds_push(&st, res);
        } else {
            fprintf(stderr, "Unexpected token\n");
            ds_free(&st);
            return 0;
        }
    }
    if (st.size != 1) { fprintf(stderr, "Invalid expression\n"); ds_free(&st); return 0; }
    *result = ds_pop(&st);
    ds_free(&st);
    return 1;
}

void print_postfix(const TokenArray *post) {
    for (int i = 0; i < post->size; ++i) {
        printf("%s ", post->data[i].text);
    }
    printf("\n");
}

int main(int argc, char **argv) {
    char buf[1024];

    if (argc >= 2) {
        buf[0] = '\0';
        for (int i = 1; i < argc; ++i) {
            strcat(buf, argv[i]);
            if (i + 1 < argc) strcat(buf, " ");
        }
    } else {
        printf("Enter an expression:\n> ");
        if (!fgets(buf, sizeof(buf), stdin)) return 0;
        size_t L = strlen(buf); if (L && buf[L-1]=='\n') buf[L-1]='\0';
    }

    TokenArray postfix = infix_to_postfix(buf);
    printf("Postfix: ");
    print_postfix(&postfix);

    double result;
    if (evaluate_postfix(&postfix, &result)) {
        printf("Result: %.10g\n", result);
    } else {
        fprintf(stderr, "Evaluation failed.\n");
    }

    ta_free(&postfix);
    return 0;
}

