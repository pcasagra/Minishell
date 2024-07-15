# Minishell
Usefull documentation for the project Minishell (creating a bash language)

1. Token Types:
Let's define token types based on the requirements you've listed:



```c
typedef enum {
    TOKEN_WORD,           // Regular words/commands
    TOKEN_SINGLE_QUOTE,   // '
    TOKEN_DOUBLE_QUOTE,   // "
    TOKEN_REDIR_IN,       // 
    TOKEN_REDIR_OUT,      // >
    TOKEN_REDIR_APPEND,   // >>
    TOKEN_HEREDOC,        // 
    TOKEN_PIPE,           // |
    TOKEN_ENV_VAR,        // $...
    TOKEN_EXIT_STATUS,    // $?
    TOKEN_AND,            // &&
    TOKEN_OR,             // ||
    TOKEN_LPAREN,         // (
    TOKEN_RPAREN,         // )
    TOKEN_WILDCARD,       // *
    TOKEN_EOF             // End of input
} TokenType;

```

2. Grammar Rules:
Now, let's define a simple grammar for your minishell. This will help in creating the abstract syntax tree.





# Grammar Rules for Minishell

```
command_line    : pipeline
                | command_line AND pipeline
                | command_line OR pipeline

pipeline        : command
                | pipeline PIPE command

command         : simple_command
                | LPAREN command_line RPAREN

simple_command  : word_list redirection_list

word_list       : word
                | word_list word

word            : WORD
                | SINGLE_QUOTE
                | DOUBLE_QUOTE
                | ENV_VAR
                | EXIT_STATUS
                | WILDCARD

redirection_list: (empty)
                | redirection_list redirection

redirection     : REDIR_IN WORD
                | REDIR_OUT WORD
                | REDIR_APPEND WORD
                | HEREDOC WORD
```


3. Abstract Syntax Tree (AST) Architecture:
For the AST, we'll need to define structures that represent each node type in our grammar. Here's a basic structure for the AST:



```c
typedef struct ASTNode ASTNode;

struct ASTNode {
    enum {
        NODE_COMMAND_LINE,
        NODE_PIPELINE,
        NODE_COMMAND,
        NODE_SIMPLE_COMMAND,
        NODE_WORD,
        NODE_REDIRECTION
    } type;
    
    union {
        struct {
            ASTNode *left;
            ASTNode *right;
            int operator;  // AND or OR
        } command_line;
        
        struct {
            ASTNode *commands;
            int command_count;
        } pipeline;
        
        struct {
            ASTNode *words;
            int word_count;
            ASTNode *redirections;
            int redirection_count;
        } simple_command;
        
        struct {
            char *value;
            int type;  // WORD, SINGLE_QUOTE, DOUBLE_QUOTE, ENV_VAR, EXIT_STATUS, WILDCARD
        } word;
        
        struct {
            int type;  // REDIR_IN, REDIR_OUT, REDIR_APPEND, HEREDOC
            char *file;
        } redirection;
    } data;
};

ASTNode *create_ast_node(int type);
void free_ast(ASTNode *node);

```

4. Recursive Descent Parsing:
To implement recursive descent parsing, you'll need to create functions for each non-terminal in your grammar. Here's a basic outline:



```c
ASTNode *parse_command_line();
ASTNode *parse_pipeline();
ASTNode *parse_command();
ASTNode *parse_simple_command();
ASTNode *parse_word_list();
ASTNode *parse_word();
ASTNode *parse_redirection_list();
ASTNode *parse_redirection();

Token current_token;
Token get_next_token();

ASTNode *parse_command_line() {
    ASTNode *node = parse_pipeline();
    
    while (current_token.type == TOKEN_AND || current_token.type == TOKEN_OR) {
        ASTNode *new_node = create_ast_node(NODE_COMMAND_LINE);
        new_node->data.command_line.left = node;
        new_node->data.command_line.operator = current_token.type;
        get_next_token();
        new_node->data.command_line.right = parse_pipeline();
        node = new_node;
    }
    
    return node;
}

// Implement other parsing functions similarly...

```

1. Implement a lexer to tokenize the input string.
2. Implement the recursive descent parser functions outlined above.
3. Create functions to execute the commands represented by the AST.
4. Implement the required built-in commands (echo, cd, pwd, export, unset, env, exit).
5. Handle environment variables and wildcards.
6. Implement input/output redirection and pipes.

I understand you'd like more detailed explanations on the grammar rules, AST structure, and recursive descent parser, along with a graphical representation. Let's go through each of these step by step.

1. Grammar Rules Explanation:

Let's break down the grammar rules:



```
command_line    : pipeline
                | command_line AND pipeline
                | command_line OR pipeline
```
- A command_line can be:
  1. A single pipeline
  2. A command_line followed by AND and another pipeline
  3. A command_line followed by OR and another pipeline

```
pipeline        : command
                | pipeline PIPE command
```
- A pipeline can be:
  1. A single command
  2. A pipeline followed by a PIPE and another command

```
command         : simple_command
                | LPAREN command_line RPAREN
```
- A command can be:
  1. A simple_command
  2. A command_line enclosed in parentheses

```
simple_command  : word_list redirection_list
```
- A simple_command is a word_list followed by a redirection_list

```
word_list       : word
                | word_list word
```
- A word_list can be:
  1. A single word
  2. A word_list followed by another word

```
word            : WORD
                | SINGLE_QUOTE
                | DOUBLE_QUOTE
                | ENV_VAR
                | EXIT_STATUS
                | WILDCARD
```
- A word can be any of these token types

```
redirection_list: (empty)
                | redirection_list redirection
```
- A redirection_list can be:
  1. Empty
  2. A redirection_list followed by a redirection

```
redirection     : REDIR_IN WORD
                | REDIR_OUT WORD
                | REDIR_APPEND WORD
                | HEREDOC WORD
```
- A redirection can be any of these combinations

2. AST Structure Explanation:

Now, let's explain the AST structure:



1. We define an ASTNode struct that can represent any node in our abstract syntax tree.

2. Each node has a type, which is an enum representing different kinds of nodes (command line, pipeline, command, etc.).

3. The node also has a union called data, which contains different structures for each node type. This allows us to store type-specific information efficiently.

4. For example:
   - A command_line node has left and right child nodes, and an operator (AND or OR).
   - A pipeline node has an array of commands and a count of how many commands are in the pipeline.
   - A simple_command node has arrays for words and redirections, along with their counts.
   - A word node has a value and a type.
   - A redirection node has a type and a file name.

5. This structure allows us to represent the entire command structure as a tree, with each node containing the necessary information for that part of the command.

3. Recursive Descent Parser Functions:

Here's an implementation of the parsing functions:



```c
ASTNode *parse_command_line() {
    ASTNode *node = parse_pipeline();
    
    while (current_token.type == TOKEN_AND || current_token.type == TOKEN_OR) {
        ASTNode *new_node = create_ast_node(NODE_COMMAND_LINE);
        new_node->data.command_line.left = node;
        new_node->data.command_line.operator = current_token.type;
        get_next_token();
        new_node->data.command_line.right = parse_pipeline();
        node = new_node;
    }
    
    return node;
}

ASTNode *parse_pipeline() {
    ASTNode *node = create_ast_node(NODE_PIPELINE);
    node->data.pipeline.commands = malloc(sizeof(ASTNode*));
    node->data.pipeline.commands[0] = parse_command();
    node->data.pipeline.command_count = 1;
    
    while (current_token.type == TOKEN_PIPE) {
        get_next_token();
        node->data.pipeline.command_count++;
        node->data.pipeline.commands = realloc(node->data.pipeline.commands, 
                                               node->data.pipeline.command_count * sizeof(ASTNode*));
        node->data.pipeline.commands[node->data.pipeline.command_count - 1] = parse_command();
    }
    
    return node;
}

ASTNode *parse_command() {
    if (current_token.type == TOKEN_LPAREN) {
        get_next_token();
        ASTNode *node = parse_command_line();
        if (current_token.type != TOKEN_RPAREN) {
            // Error handling
        }
        get_next_token();
        return node;
    } else {
        return parse_simple_command();
    }
}

ASTNode *parse_simple_command() {
    ASTNode *node = create_ast_node(NODE_SIMPLE_COMMAND);
    node->data.simple_command.words = parse_word_list();
    node->data.simple_command.redirections = parse_redirection_list();
    return node;
}

ASTNode *parse_word_list() {
    ASTNode *node = create_ast_node(NODE_WORD);
    node->data.word.value = strdup(current_token.value);
    node->data.word.type = current_token.type;
    get_next_token();
    
    while (is_word_token(current_token.type)) {
        // Add word to list
        get_next_token();
    }
    
    return node;
}

ASTNode *parse_redirection_list() {
    ASTNode *node = NULL;
    
    while (is_redirection_token(current_token.type)) {
        ASTNode *redir_node = create_ast_node(NODE_REDIRECTION);
        redir_node->data.redirection.type = current_token.type;
        get_next_token();
        if (!is_word_token(current_token.type)) {
            // Error handling
        }
        redir_node->data.redirection.file = strdup(current_token.value);
        get_next_token();
        
        // Add redir_node to list
    }
    
    return node;
}

// Helper functions
int is_word_token(TokenType type) {
    return type == TOKEN_WORD || type == TOKEN_SINGLE_QUOTE || type == TOKEN_DOUBLE_QUOTE ||
           type == TOKEN_ENV_VAR || type == TOKEN_EXIT_STATUS || type == TOKEN_WILDCARD;
}

int is_redirection_token(TokenType type) {
    return type == TOKEN_REDIR_IN || type == TOKEN_REDIR_OUT || 
           type == TOKEN_REDIR_APPEND || type == TOKEN_HEREDOC;
}

```

4. Graphical Representation:

Here's a simplified graphical representation of the AST and parsing functions:



```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 800 600">
  <defs>
    <marker id="arrowhead" markerWidth="10" markerHeight="7" refX="0" refY="3.5" orient="auto">
      <polygon points="0 0, 10 3.5, 0 7" />
    </marker>
  </defs>
  
  <!-- AST Structure -->
  <rect x="50" y="50" width="300" height="200" fill="none" stroke="black" />
  <text x="200" y="70" text-anchor="middle">AST Structure</text>
  <text x="70" y="100">- Node Types</text>
  <text x="70" y="130">- Command Line</text>
  <text x="70" y="160">- Pipeline</text>
  <text x="70" y="190">- Simple Command</text>
  <text x="70" y="220">- Word / Redirection</text>

  <!-- Parsing Functions -->
  <rect x="450" y="50" width="300" height="200" fill="none" stroke="black" />
  <text x="600" y="70" text-anchor="middle">Parsing Functions</text>
  <text x="470" y="100">parse_command_line()</text>
  <text x="470" y="130">parse_pipeline()</text>
  <text x="470" y="160">parse_command()</text>
  <text x="470" y="190">parse_simple_command()</text>
  <text x="470" y="220">parse_word_list() / parse_redirection_list()</text>

  <!-- Arrows -->
  <line x1="350" y1="150" x2="450" y2="150" stroke="black" marker-end="url(#arrowhead)" />
  <line x1="600" y1="250" x2="600" y2="300" stroke="black" marker-end="url(#arrowhead)" />
  <line x1="200" y1="250" x2="200" y2="300" stroke="black" marker-end="url(#arrowhead)" />

  <!-- Input -->
  <rect x="300" y="300" width="200" height="50" fill="none" stroke="black" />
  <text x="400" y="330" text-anchor="middle">Input String</text>

  <!-- Tokenizer -->
  <rect x="300" y="400" width="200" height="50" fill="none" stroke="black" />
  <text x="400" y="430" text-anchor="middle">Tokenizer</text>

  <!-- Final AST -->
  <rect x="300" y="500" width="200" height="50" fill="none" stroke="black" />
  <text x="400" y="530" text-anchor="middle">Final AST</text>

  <!-- Connecting Arrows -->
  <line x1="400" y1="350" x2="400" y2="400" stroke="black" marker-end="url(#arrowhead)" />
  <line x1="400" y1="450" x2="400" y2="500" stroke="black" marker-end="url(#arrowhead)" />
</svg>

```

This diagram shows:
1. The AST structure on the left
2. The parsing functions on the right
3. How the input string is tokenized and then parsed into the final AST

The arrows indicate the flow of data and the relationships between different components of the system. The parsing functions work together to build the AST structure based on the tokenized input.

Is there any part of this explanation you'd like me to elaborate on further?
