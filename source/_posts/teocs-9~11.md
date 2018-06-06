---
title: teocs-9~11
date: 2016-10-10 15:43:23
categories: programming
tags: course
---

### 从零开始构建现代计算机

#### 九.高级语言

**基本类型**申明时会在内存中分配对应的内存单元

**对象类型**申明时实际创建一个指向该对象的引用变量，内存单元只有在调用构造函数时才会分配

<!--more-->

#### **十~十一.编译器**

##### 十：

##### **1.语法分析**—字元化(tokenizing)模块将输入的字符分组成语言原子元素；语法分析(parsing)模块将语言原子元素集合同语法规则相匹配

##### 词法分析

**上下文无关语法**

**语法分析树**

LL(0)语法，每当一个非终结符有多重可选择导出规则时，其第一个字元足以确定该非终结符所属的表达式类型，而不会出现不确定的情况。



测试过的代码（以为没问题，无聊测了下，BUG不少，以后还是保持感觉良好）

```python
#!usr/bin/env python
# -*-coding:utf-8-*-
"""
author:wubaichuan

"""
import re
from xml.sax.saxutils import escape


class JackTokenizer(object):
    SYMBOL = ('{', '}', '(', ')', '[', ']', '.', ',', ';', '+', '-', '*', '/', '&', '|', '<', '>', '=', '~')
    KEYWORD = ('class', 'constructor', 'function', 'method', 'field', 'static', 'var', 'int', 'char', 'boolean',
               'void', 'true', 'false', 'null', 'this', 'let', 'do', 'if', 'else', 'while', 'return')

    def __init__(self):
        self.token_list = []
        self.token = None

    def constructor(self, file_uri):
        self.token_list = []
        with open(file_uri, 'rb') as f:
            temp_token = f.read(1)
            while temp_token:
                if temp_token.isalpha() or temp_token.isdigit() or temp_token == '_':
                    self.token_list.append(temp_token)
                    temp_token = f.read(1)
                    while temp_token.isalpha() or temp_token.isdigit() or temp_token == '_':
                        self.token_list[-1] += temp_token
                        temp_token = f.read(1)
                    f.seek(-1, 1)
                elif temp_token == '/':
                    temp_token = f.read(1)
                    if temp_token == '/':
                        while f.read(1) != '\n':
                            continue
                    elif temp_token == '*':
                        while True:
                            temp_token = f.read(1)
                            if temp_token == '*':
                                if f.read(1) == '/':
                                    break
                                else:
                                    f.seek(-1, 1)
                    else:
                        f.seek(-1, 1)
                        self.token_list.append('/')
                elif temp_token in JackTokenizer.SYMBOL:
                    self.token_list.append(temp_token)
                elif temp_token == '"':
                    self.token_list.append(temp_token)
                    temp_token = None
                    while temp_token != '"':
                        temp_token = f.read(1)
                        self.token_list[-1] += temp_token
                temp_token = f.read(1)
        print self.token_list
        self.token_list.reverse()
        self.advance()

    def has_more_tokens(self):
        return self.token_list != []

    def advance(self):
        self.token = self.token_list.pop()

    @property
    def token_type(self):
        if self.token in JackTokenizer.KEYWORD:
            return 'KEYWORD'
        elif self.token in JackTokenizer.SYMBOL:
            return 'SYMBOL'
        elif self.token[0].isdigit():
            return 'INT_CONST'
        elif self.token[0] == '"':
            return 'STRING_CONST'
        else:
            return 'IDENTIFIER'

    @property
    def keyword(self):
        return self.token

    @property
    def symbol(self):
        return self.token

    @property
    def int_val(self):
        return self.token

    @property
    def string_val(self):
        return self.token.lstrip('"').strip('"')

    @property
    def identifier(self):
        return self.token


class CompilationEngine(object):
    TYPE_MAP = {'KEYWORD': 'keyword', 'SYMBOL': 'symbol', 'INT_CONST': 'integerConstant',
                'STRING_CONST': 'stringConstant', 'IDENTIFIER': 'identifier'}

    def __init__(self, in_file_uri, out_file_uri):
        self.tokenizer = JackTokenizer()
        self.tokenizer.constructor(in_file_uri)
        self.out_file = file(out_file_uri, 'w')

    def __del__(self):
        self.out_file.close()

    def write_end_xml(self, tag):
        self.out_file.write('<{0}>{1}</{0}>\n'.format(
            tag, escape(
                self.tokenizer.token if self.tokenizer.token_type != 'STRING_CONST' else self.tokenizer.string_val)))
        if self.tokenizer.has_more_tokens():
            self.tokenizer.advance()

    def compile_type(self):
        self.write_end_xml(CompilationEngine.TYPE_MAP[self.tokenizer.token_type])

    def compile_class(self):
        self.out_file.write('<class>\n')
        self.write_end_xml('keyword')
        self.write_end_xml('identifier')
        self.write_end_xml('symbol')
        while self.tokenizer.token in ['static', 'field']:
            self.compile_class_var_dec()
        while self.tokenizer.token in ['constructor', 'function', 'method']:
            self.compile_subroutine()
        self.write_end_xml('symbol')
        self.out_file.write('</class>\n')

    def compile_class_var_dec(self):
        self.out_file.write('<classVarDec>\n')
        self.write_end_xml('keyword')
        self.compile_type()
        self.write_end_xml('identifier')
        while self.tokenizer.token != ';':
            self.write_end_xml('symbol')
            self.write_end_xml('identifier')
        self.write_end_xml('symbol')
        self.out_file.write('</classVarDec>\n')

    def compile_subroutine(self):
        self.out_file.write('<subroutineDec>\n')
        self.write_end_xml('keyword')
        self.compile_type()
        self.write_end_xml('identifier')
        self.write_end_xml('symbol')
        self.compile_parameter_list()
        self.write_end_xml('symbol')
        self.compile_subroutine_body()
        self.out_file.write('</subroutineDec>\n')

    def compile_subroutine_body(self):
        self.out_file.write('<subroutineBody>\n')
        self.write_end_xml('symbol')
        while self.tokenizer.token == 'var':
            self.compile_var_dec()
        self.compile_statements()
        self.write_end_xml('symbol')
        self.out_file.write('</subroutineBody>\n')

    def compile_parameter_list(self):
        self.out_file.write('<parameterList>\n')
        if self.tokenizer.token != ')':
            self.compile_type()
            self.write_end_xml('identifier')
            while self.tokenizer.token == ',':
                self.write_end_xml('symbol')
                self.compile_type()
                self.write_end_xml('identifier')
        self.out_file.write('</parameterList>\n')

    def compile_var_dec(self):
        self.out_file.write('<varDec>\n')
        self.write_end_xml('keyword')
        self.compile_type()
        self.write_end_xml('identifier')
        while self.tokenizer.token != ';':
            self.write_end_xml('symbol')
            self.write_end_xml('identifier')
        self.write_end_xml('symbol')
        self.out_file.write('</varDec>\n')

    def compile_statements(self):
        self.out_file.write('<statements>\n')
        statement_map = {'if': self.compile_if, 'let': self.compile_let, 'while': self.compile_while,
                         'do': self.compile_do, 'return': self.compile_return}
        while self.tokenizer.token in statement_map.keys():
            statement_map[self.tokenizer.token]()
        self.out_file.write('</statements>\n')

    def compile_do(self):
        self.out_file.write('<doStatement>\n')
        self.write_end_xml('keyword')
        self.compile_subroutine_call()
        self.write_end_xml('symbol')
        self.out_file.write('</doStatement>\n')

    def compile_subroutine_call(self):
        self.write_end_xml('identifier')
        while self.tokenizer.token == '.':
            self.write_end_xml('symbol')
            self.write_end_xml('identifier')
        self.write_end_xml('symbol')
        self.compile_expression_list()
        self.write_end_xml('symbol')

    def compile_let(self):
        self.out_file.write('<letStatement>\n')
        self.write_end_xml('keyword')
        self.write_end_xml('identifier')
        while self.tokenizer.token == '[':
            self.write_end_xml('symbol')
            self.compile_expression()
            self.write_end_xml('symbol')
        self.write_end_xml('symbol')
        self.compile_expression()
        self.write_end_xml('symbol')
        self.out_file.write('</letStatement>\n')

    def compile_while(self):
        self.out_file.write('<whileStatement>\n')
        self.write_end_xml('keyword')
        self.write_end_xml('symbol')
        self.compile_expression()
        self.write_end_xml('symbol')
        self.write_end_xml('symbol')
        self.compile_statements()
        self.write_end_xml('symbol')
        self.out_file.write('</whileStatement>\n')

    def compile_return(self):
        self.out_file.write('<returnStatement>\n')
        self.write_end_xml('keyword')
        if self.tokenizer.token != ';':
            self.compile_expression()
        self.write_end_xml('symbol')
        self.out_file.write('</returnStatement>\n')

    def compile_if(self):
        self.out_file.write('<ifStatement>\n')
        self.write_end_xml('keyword')
        self.write_end_xml('symbol')
        self.compile_expression()
        self.write_end_xml('symbol')
        self.write_end_xml('symbol')
        self.compile_statements()
        self.write_end_xml('symbol')
        if self.tokenizer.token == 'else':
            self.write_end_xml('keyword')
            self.write_end_xml('symbol')
            self.compile_statements()
            self.write_end_xml('symbol')
        self.out_file.write('</ifStatement>\n')

    def compile_expression(self):
        self.out_file.write('<expression>\n')
        self.compile_term()
        while self.tokenizer.token in "+-*/&|<>=":
            self.write_end_xml('symbol')
            self.compile_term()
        self.out_file.write('</expression>\n')

    def compile_term(self):
        self.out_file.write('<term>\n')
        if self.tokenizer.token in '-~':
            self.write_end_xml('symbol')
            self.compile_term()
        elif self.tokenizer.token == '(':
            self.write_end_xml('symbol')
            self.compile_expression()
            self.write_end_xml('symbol')
        else:
            store_token = self.tokenizer.token
            self.tokenizer.advance()
            if self.tokenizer.token in '(.':
                self.tokenizer.token_list.append(self.tokenizer.token)
                self.tokenizer.token = store_token
                self.compile_subroutine_call()
            else:
                self.tokenizer.token_list.append(self.tokenizer.token)
                self.tokenizer.token = store_token
                self.compile_type()
                if self.tokenizer.token == '[':
                    self.write_end_xml('symbol')
                    self.compile_expression()
                    self.write_end_xml('symbol')
        self.out_file.write('</term>\n')

    def compile_expression_list(self):
        self.out_file.write('<expression>\n')
        if self.tokenizer.token != ')':
            self.compile_expression()
            while self.tokenizer.token == ',':
                self.write_end_xml('symbol')
                self.compile_expression()
        self.out_file.write('</expression>\n')


def analyze(uri=None):
    compile_obj = CompilationEngine(uri, uri.replace('.jack', '1.xml'))
    while compile_obj.tokenizer.has_more_tokens():
        compile_obj.compile_class()

if __name__ == '__main__':
    analyze('/Users/alian/Desktop/nand2tetris/projects/10/ExpressionlessSquare/SquareGame.jack')



```

##### 十一

**数据的翻译**

**命令的翻译**

```python
#!usr/bin/env python
# -*-coding:utf-8-*-
"""
author:wubaichuan

"""


class JackTokenizer(object):
    SYMBOL = ('{', '}', '(', ')', '[', ']', '.', ',', ';', '+', '-', '*', '/', '&', '|', '<', '>', '=', '~')
    KEYWORD = ('class', 'constructor', 'function', 'method', 'field', 'static', 'var', 'int', 'char', 'boolean',
               'void', 'true', 'false', 'null', 'this', 'let', 'do', 'if', 'else', 'while', 'return')

    def __init__(self):
        self.token_list = []
        self.token = None

    def constructor(self, file_uri):
        self.token_list = []
        with open(file_uri, 'rb') as f:
            temp_token = f.read(1)
            while temp_token:
                if temp_token.isalpha() or temp_token.isdigit() or temp_token == '_':
                    self.token_list.append(temp_token)
                    temp_token = f.read(1)
                    while temp_token.isalpha() or temp_token.isdigit() or temp_token == '_':
                        self.token_list[-1] += temp_token
                        temp_token = f.read(1)
                    f.seek(-1, 1)
                elif temp_token == '/':
                    temp_token = f.read(1)
                    if temp_token == '/':
                        while f.read(1) != '\n':
                            continue
                    elif temp_token == '*':
                        while True:
                            temp_token = f.read(1)
                            if temp_token == '*':
                                if f.read(1) == '/':
                                    break
                                else:
                                    f.seek(-1, 1)
                    else:
                        f.seek(-1, 1)
                        self.token_list.append('/')
                elif temp_token in JackTokenizer.SYMBOL:
                    self.token_list.append(temp_token)
                elif temp_token == '"':
                    self.token_list.append(temp_token)
                    temp_token = None
                    while temp_token != '"':
                        temp_token = f.read(1)
                        self.token_list[-1] += temp_token
                temp_token = f.read(1)
        print self.token_list
        self.token_list.reverse()
        self.advance()

    def has_more_tokens(self):
        return self.token_list != []

    def advance(self):
        self.token = self.token_list.pop()

    @property
    def token_type(self):
        if self.token in JackTokenizer.KEYWORD:
            return 'KEYWORD'
        elif self.token in JackTokenizer.SYMBOL:
            return 'SYMBOL'
        elif self.token[0].isdigit():
            return 'INT_CONST'
        elif self.token[0] == '"':
            return 'STRING_CONST'
        else:
            return 'IDENTIFIER'

    @property
    def keyword(self):
        return self.token

    @property
    def symbol(self):
        return self.token

    @property
    def int_val(self):
        return self.token

    @property
    def string_val(self):
        return self.token.lstrip('"').strip('"')

    @property
    def identifier(self):
        return self.token


class SymbolTable(object):
    CLASS_SCOPE = ['static', 'field']
    SUBROUTINE_SCOPE = ['argument', 'var']
    ALL_SCOPE = CLASS_SCOPE + SUBROUTINE_SCOPE

    def __init__(self):
        self._symbol = {'static': {}, 'field': {}, 'argument': {}, 'var': {}}
        self.index = {'static': 0, 'field': 0, 'argument': 0, 'var': 0}

    def start_subroutine(self):
        self._symbol['var'].clear()
        self._symbol['argument'].clear()
        self.index['argument'] = self.index['var'] = 0

    def define(self, name, type_, kind):
        self._symbol[kind][name] = (type_, self.index[kind])
        self.index[kind] += 1

    def var_count(self, kind):
        return len(self._symbol[kind])

    def kind_of(self, name):
        for i in SymbolTable.ALL_SCOPE:
            if self._symbol[i].get(name) is not None:
                return i

    def type_of(self, name):
        for i in SymbolTable.ALL_SCOPE:
            if self._symbol[i].get(name) is not None:
                return self._symbol[i][name][0]

    def index_of(self, name):
        for i in SymbolTable.ALL_SCOPE:
            if self._symbol[i].get(name) is not None:
                return self._symbol[i][name][1]


class VMWriter(object):
    def __init__(self, file_uri):
        self.file = open(file_uri, 'w')

    def __del__(self):
        self.file.close()

    def write_pop(self, segment, index):
        if segment == 'var':
            segment = 'local'
        self.file.write('pop {} {}\n'.format(segment, index))

    def write_push(self, segment, index):
        if segment == 'var':
            segment = 'local'
        self.file.write('push {} {}\n'.format(segment, index))

    def write_arithmetic(self, command):
        self.file.write('{}\n'.format(command))

    def write_label(self, label):
        self.file.write('label {}\n'.format(label))

    def write_goto(self, label):
        self.file.write('goto {}\n'.format(label))

    def write_if(self, label):
        self.file.write('if-goto {}\n'.format(label))

    def write_call(self, name, nargs):
        self.file.write('call {} {}\n'.format(name, nargs))

    def write_function(self, name, nargs):
        self.file.write('function {} {}\n'.format(name, nargs))

    def write_return(self):
        self.file.write('return\n')


class CompilationEngine(object):
    TYPE_MAP = {'KEYWORD': 'keyword', 'SYMBOL': 'symbol', 'INT_CONST': 'integerConstant',
                'STRING_CONST': 'stringConstant', 'IDENTIFIER': 'identifier'}

    OPS = {'=': 'eq', '+': 'add', '-': 'sub', '&': 'and', '|': 'or', '~': 'not', '<': 'lt', '>': 'gt'}

    def __init__(self, in_file_uri, out_file_uri):
        self.tokenizer = JackTokenizer()
        self.tokenizer.constructor(in_file_uri)
        self.symbols = SymbolTable()
        self.vm = VMWriter(out_file_uri)
        self.class_name = ''
        self._tmp_type = ''
        self._tmp_kind = ''
        self.if_num = 0
        self.while_num = 0

    def abort_token(self, need_token, end_of_class=False):
        if isinstance(need_token, basestring):
            assert self.tokenizer.token == need_token, "wrong format of jack file"
        else:
            assert self.tokenizer.token in need_token, "wrong format of jack file"
        if not end_of_class or (end_of_class and self.tokenizer.token_list):
            self.tokenizer.advance()

    @property
    def current_token(self):
        tmp = self.tokenizer.token
        self.tokenizer.advance()
        return tmp

    def define_symbol(self):
        self.symbols.define(self.current_token, self._tmp_type, self._tmp_kind)

    def compile_class(self):
        self.abort_token('class')
        self.class_name = self.current_token
        self.abort_token('{')
        while self.tokenizer.token in ['static', 'field']:
            self.compile_class_var_dec()
        while self.tokenizer.token in ['constructor', 'function', 'method']:
            self.compile_subroutine()
        self.abort_token('}', True)

    def compile_class_var_dec(self):
        self._tmp_kind = self.current_token
        self._tmp_type = self.current_token
        self.define_symbol()
        while self.tokenizer.token == ',':
            self.abort_token(',')
            self.define_symbol()
        self.abort_token(';')

    def compile_subroutine(self):
        kind = self.current_token
        type_ = self.current_token
        name = self.current_token
        self.symbols.start_subroutine()
        self.abort_token('(')
        self.compile_parameter_list(kind)
        self.abort_token(')')
        self.compile_subroutine_body(kind, name)

    def compile_subroutine_body(self, func_kind, func_name):
        self.abort_token('{')
        while self.tokenizer.token == 'var':
            self.compile_var_dec()
        self.vm.write_function('{0}.{1}'.format(self.class_name, func_name), self.symbols.var_count('var'))
        if func_kind == 'constructor':
            self.vm.write_push('constant', self.symbols.var_count('field'))
            self.vm.write_call('Memory.alloc', 1)
            self.vm.write_pop('pointer', 0)
        elif func_kind == 'method':
            self.vm.write_push('argument', 0)
            self.vm.write_pop('pointer', 0)
        self.compile_statements()
        self.abort_token('}')

    def compile_parameter_list(self, category):
        self._tmp_kind = 'argument'
        if category == 'method':
            self.symbols.define('this', None, self._tmp_kind)
        if self.tokenizer.token != ')':
            self._tmp_type = self.current_token
            self.define_symbol()
            while self.tokenizer.token != ')':
                self.abort_token(',')
                self._tmp_type = self.current_token
                self.define_symbol()

    def compile_var_dec(self):
        self.abort_token('var')
        self._tmp_kind = 'var'
        self._tmp_type = self.current_token
        self.define_symbol()
        while self.tokenizer.token != ';':
            self.abort_token(',')
            self.define_symbol()
        self.abort_token(';')

    def compile_statements(self):
        statement_map = {'if': self.compile_if, 'let': self.compile_let, 'while': self.compile_while,
                         'do': self.compile_do, 'return': self.compile_return}
        while self.tokenizer.token in statement_map.keys():
            statement_map[self.tokenizer.token]()

    def compile_do(self):
        self.abort_token('do')
        self.compile_subroutine_call()
        self.vm.write_pop('temp', 0)
        self.abort_token(';')

    def compile_subroutine_call(self):
        count = 0
        name = self.current_token
        kind, index = self.symbols.kind_of(name), self.symbols.index_of(name)
        if self.tokenizer.token == '.':
            if kind in ['field', 'var', 'static']:
                self.vm.write_push(kind, index)
                count = 1
            pre_fix = self.symbols.type_of(name)
            pre_fix = pre_fix if pre_fix else name
            self.abort_token('.')
            func_name = self.current_token
        else:
            self.vm.write_push('pointer', 0)
            pre_fix = self.class_name
            func_name = name
            count = 1
        self.abort_token('(')
        count += self.compile_expression_list()
        self.abort_token(')')
        self.vm.write_call('{0}.{1}'.format(pre_fix, func_name), count)

    def compile_let(self):
        self.abort_token('let')
        name = self.current_token
        kind, type_, index = self.symbols.kind_of(name), self.symbols.type_of(name), self.symbols.index_of(name)
        if self.tokenizer.token == '[':
            self.abort_token('[')
            self.compile_expression()
            self.abort_token(']')
            self.vm.write_push(kind, index)
            self.vm.write_arithmetic('add')
            self.abort_token('=')
            self.compile_expression()
            self.vm.write_pop('temp', 0)
            self.vm.write_pop('pointer', 1)
            self.vm.write_push('temp', 0)
            self.vm.write_pop('that', 0)
        else:
            self.abort_token('=')
            self.compile_expression()
            self.vm.write_pop(kind, index)
        self.abort_token(';')

    def compile_while(self):
        self.abort_token('while')
        fit_label = 'W_FIT%d' % self.while_num
        end_label = 'W_END%d' % self.while_num
        self.while_num += 1
        self.vm.write_label(fit_label)
        self.abort_token('(')
        self.compile_expression()
        self.abort_token(')')
        self.vm.write_arithmetic('not')
        self.vm.write_if(end_label)
        self.abort_token('{')
        self.compile_statements()
        self.abort_token('}')
        self.vm.write_goto(fit_label)
        self.vm.write_label(end_label)

    def compile_return(self):
        self.abort_token('return')
        if self.tokenizer.token != ';':
            self.compile_expression()
        else:
            self.vm.write_push('constant', 0)
        self.abort_token(';')
        self.vm.write_return()

    def compile_if(self):
        self.abort_token('if')
        fit_label = 'I_FIT%d' % self.if_num
        else_label = 'I_ELSE%d' % self.if_num
        end_label = 'I_END%d' % self.if_num
        self.if_num += 1
        self.abort_token('(')
        self.compile_expression()
        self.abort_token(')')
        self.vm.write_if(fit_label)
        self.vm.write_goto(else_label)
        self.vm.write_label(fit_label)
        self.abort_token('{')
        self.compile_statements()
        self.abort_token('}')
        if self.tokenizer.token == 'else':
            self.vm.write_goto(end_label)
        self.vm.write_label(else_label)
        if self.tokenizer.token == 'else':
            self.abort_token('else')
            self.abort_token('{')
            self.compile_statements()
            self.abort_token('}')
            self.vm.write_label(end_label)

    def compile_expression(self):
        self.compile_term()
        while self.tokenizer.token in "+-*/&|<>=":
            token = self.current_token
            self.compile_term()
            if token == '/':
                self.vm.write_call('Math.divide', 2)
            elif token == '*':
                self.vm.write_call('Math.multiply', 2)
            else:
                self.vm.write_arithmetic(CompilationEngine.OPS[token])

    def compile_term(self):
        if self.tokenizer.token_type == 'INT_CONST':
            self.vm.write_push('constant', self.current_token)
        elif self.tokenizer.token_type == 'STRING_CONST':
            token = self.current_token.strip('"').lstrip('"')
            self.vm.write_push('constant', len(token))
            self.vm.write_call('String.new', 1)
            for i in token:
                self.vm.write_push('constant', ord(i))
                self.vm.write_call('String.appendChar', 2)
        elif self.tokenizer.token_type == 'KEYWORD':
            token = self.current_token
            if token in ['false', 'null']:
                self.vm.write_push('constant', 0)
            elif token == 'true':
                self.vm.write_push('constant', 0)
                self.vm.write_arithmetic('not')
            elif token == 'this':
                self.vm.write_push('pointer', 0)
            else:
                raise ValueError('Wrong term')
        elif self.tokenizer.token_type == 'SYMBOL':
            token = self.current_token
            if token == '(':
                self.compile_expression()
                self.abort_token(')')
            elif token in '-~':
                temp_dict = {'-': 'neg', '~': 'not'}
                self.compile_term()
                self.vm.write_arithmetic(temp_dict[token])
            else:
                raise ValueError('Wrong term')
        elif self.tokenizer.token_type == 'IDENTIFIER':
            store_token = self.current_token
            kind = self.symbols.kind_of(store_token)
            index = self.symbols.index_of(store_token)
            if self.tokenizer.token in '(.':
                self.tokenizer.token_list.append(self.tokenizer.token)
                self.tokenizer.token = store_token
                self.compile_subroutine_call()
            elif self.tokenizer.token == '[':
                self.abort_token('[')
                self.compile_expression()
                self.abort_token(']')
                self.vm.write_push(kind, index)
                self.vm.write_arithmetic('add')
                self.vm.write_pop('pointer', 1)
                self.vm.write_push('that', 0)
            else:
                self.vm.write_push(kind, index)
        else:
            raise ValueError('Wrong term')

    def compile_expression_list(self):
        count = 0
        if self.tokenizer.token != ')':
            self.compile_expression()
            count += 1
            while self.tokenizer.token == ',':
                self.abort_token(',')
                self.compile_expression()
                count += 1
        return count


def analyze(uri):
    compile_obj = CompilationEngine(uri, uri.replace('.jack', '1.vm'))
    while compile_obj.tokenizer.has_more_tokens():
        compile_obj.compile_class()

if __name__ == '__main__':
    analyze('/Users/alian/Desktop/nand2tetris/projects/11/ConvertToBin/Main.jack')
```

