---
title: teocs-7~8
date: 2016-10-08 17:12:42
categories: programming
tags: course
---

### 从零开始构建现代计算机

<!--more-->

#### 六.虚拟机

目的：减少编译器编译的高级语言与编译之后的机器语言间的依赖性（JAVA,C#）

虚拟机范型：略

一.堆栈运算

二.程序控制

项目：（应该没问题，没在仿真器跑）

<!--more-->

```python
#!usr/bin/env python
# -*-coding:utf-8-*-
"""
author:wubaichuan

"""
import os


class Parser(object):
    COMMAND_TYPE_DICT = {'add': 'C_ARITHMETIC', 'sub': 'C_ARITHMETIC',
                         'neg': 'C_ARITHMETIC', 'eq': 'C_ARITHMETIC',
                         'gt': 'C_ARITHMETIC', 'lt': 'C_ARITHMETIC',
                         'and': 'C_ARITHMETIC', 'or': 'C_ARITHMETIC',
                         'not': 'C_ARITHMETIC',
                         'pop': 'C_POP', 'push': 'C_PUSH',
                         'label': 'C_LABEL', 'goto': 'C_GOTO',
                         'if-goto': 'C_IF', 'function': 'C_FUNCTION',
                         'return': 'C_RETURN', 'call': 'C_CALL'}

    def __init__(self, file_uri):
        self.file_uri = file_uri
        self.vm_content = self.__content()
        self.line = '//'

    def __content(self):
        if not os.path.exists(self.file_uri):
            raise NameError("file does not exist")
        temp = []
        if os.path.isfile(self.file_uri):
            with open(self.file_uri, 'r') as f:
                temp = f.readlines()
        else:
            files = [f for f in os.listdir(self.file_uri) if os.path.isfile(os.path.join(self.file_uri, f))]
            for vm_file in files:
                with open(vm_file, 'r') as f:
                    temp.extend(f.readlines())
        return temp

    def has_more_commands(self):
        return self.vm_content != []

    def advance(self):
        self.line = self.vm_content.pop(0).lstrip().strip()
        while not self.line or self.line[:2] == '//':
            self.line = self.vm_content.pop(0).lstrip()
        if '//' in self.line:
            self.line = self.line.split('//')[0].strip()

    @property
    def command_type(self):
        args = self.line.split()
        return Parser.COMMAND_TYPE_DICT[args[0]]

    @property
    def arg_1(self):
        temp_type = self.command_type
        args = self.line.split()
        if temp_type == 'C_ARITHMETIC':
            return args[0]
        elif temp_type == 'C_RETURN':
            return None
        else:
            return args[1]

    @property
    def arg_2(self):
        if self.command_type in ['C_PUSH', 'C_POP', 'C_FUNCTION', 'C_CALL']:
            return int(self.line.split()[-1])


class CodeWriter(object):
    RAM_MAP = {'this': 'THIS', 'that': 'THAT', 'local': 'LCL', 'argument': 'ARG'}
    ARITH_ASM = {'add': '@SP\nM=M-1\nA=M\nD=M\n@SP\nM=M-1\nA=M\nM=D+M\n@SP\nM=M+1\n',
                 'sub': '@SP\nM=M-1\nA=M\nD=M\n@SP\nM=M-1\nA=M\nM=M-D\n@SP\nM=M+1\n',
                 'neg': '@SP\nM=M-1\nA=M\nM=-M\n@SP\nM=M+1\n',
                 'eq': '@SP\nM=M-1\nA=M\nD=M\n@SP\nM=M-1\nA=M\nD=M-D\n' +
                       '@FIT{0}\nD;JEQ\nD=0\n@CONTINUE{0}\n0;JMP\n(FIT{0})\nD=-1\n(CONTINUE{0})\n' +
                       '@SP\nA=M\nM=D\n@SP\nM=M+1\n',
                 'gt': '@SP\nM=M-1\nA=M\nD=M\n@SP\nM=M-1\nA=M\nD=M-D\n' +
                       '@FIT{0}\nD;JGT\nD=0\n@CONTINUE{0}\n0;JMP\n(FIT{0})\nD=-1\n(CONTINUE{0})\n' +
                       '@SP\nA=M\nM=D\n@SP\nM=M+1\n',
                 'lt': '@SP\nM=M-1\nA=M\nD=M\n@SP\nM=M-1\nA=M\nD=M-D\n' +
                       '@FIT{0}\nD;JLT\nD=0\n@CONTINUE{0}\n0;JMP\n(FIT{0})\nD=-1\n(CONTINUE{0})\n' +
                       '@SP\nA=M\nM=D\n@SP\nM=M+1\n',
                 'and': '@SP\nM=M-1\nA=M\nD=M\n@SP\nM=M-1\nA=M\nM=D&M\n@SP\nM=M+1\n',
                 'or': '@SP\nM=M-1\nA=M\nD=M\n@SP\nM=M-1\nA=M\nM=D|M\n@SP\nM=M+1\n',
                 'not': '@SP\nM=M-1\nA=M\nM=!M\n@SP\nM=M+1\n'}

    def __init__(self, file_uri):
        self.filename = ''
        self.file = open(file_uri, 'w')
        self.count = 0
        self.back_count = 0
        self.env = ['']  # 不知对不对,对照Google的参考没有重新命名label
        self.write_init()

    def set_filename(self, filename):
        self.filename = os.path.basename(filename).strip('.vm')

    def write_init(self):
        self.file.write('@256\nD=A\n@SP\nM=D\n')
        self.write_call('Sys.init', 0)

    def write_arithmetic(self, command):
        self.file.write(CodeWriter.ARITH_ASM[command].format(self.count))
        self.count += 1

    def write_push_pop(self, command, segment, index):
        if command == 'C_PUSH':
            if segment == 'constant':
                self.file.write('@{0}\nD=A\n@SP\nA=M\nM=D\n@SP\nM=M+1\n'.format(index))
            elif segment in ['local', 'argument', 'this', 'that']:
                self.file.write('@{0}\nD=M\n@{1}\nA=D+A\nD=M\n@SP\nA=M\nM=D\n@SP\nM=M+1\n'.format(
                    CodeWriter.RAM_MAP[segment], index))
            elif segment == 'pointer':  # pointer 0,1 分别对应RAM[3],RAM[4]即目前this和that的基址
                self.file.write('@{0}\nD=M\n@SP\nA=M\nM=D\n@SP\nM=M+1\n'.format(index + 2))
            elif segment == 'temp':
                self.file.write('@{0}\nD=M\n@SP\nA=M\nM=D\n@SP\nM=M+1\n'.format(index + 5))
            elif segment == 'static':
                self.file.write('@{0}.{1}\nD=M\n@SP\nA=M\nM=D\n@SP\nM=M+1\n'.format(self.filename, index))

        elif command == 'C_POP':
            if segment in ['local', 'argument', 'this', 'that']:
                self.file.write('@{0}\nD=M\n@{1}\nD=D+A\n@15\nM=D\n@SP\nM=M-1\nA=M\nD=M\n@15\nA=M\nM=D\n'.format(
                    CodeWriter.RAM_MAP[segment], index))

            elif segment == 'pointer':
                self.file.write('@SP\nM=M-1\nA=M\nD=M\n@{0}\nM=D\n'.format(index + 2))

            elif segment == 'temp':
                self.file.write('@SP\nM=M-1\nA=M\nD=M\n@{0}\nM=D\n'.format(index + 5))

            elif segment == 'static':
                self.file.write('@SP\nM=M-1\nA=M\nD=M\n@{0}.{1}\nM=D\n'.format(self.filename, index))

    def write_label(self, label):
        self.file.write('({0})\n'.format(self.env[-1] + label))  # 生成'函数名$label'的label

    def write_goto(self, label):
        self.file.write('@{0}\n0;JMP\n'.format(self.env[-1] + label))

    def write_if(self, label):
        self.file.write('@SP\nM=M-1\nA=M\nD=M\n@{0}\nD;JNE\n'.format(self.env[-1] + label))

    def write_call(self, func_name, num):
        self.file.write('@RETURN_ADR{0}\nD=A\n@SP\nA=M\nM=D\n@SP\nM=M+1\n'.format(self.back_count))  # 将返回的地址压入堆栈
        for mem in ['LCL', 'ARG', 'THIS', 'THAT']:
            self.file.write('@{0}\nD=M\n@SP\nA=M\nM=D\n@SP\nM=M+1\n'.format(mem))  # 将调用函数的种种压入堆栈
        self.file.write('@{0}\nD=A\n@SP\nD=M-D\n@ARG\nM=D\n'.format(5 + num))  # 重置堆栈顶
        self.file.write('@SP\nD=M\n@LCL\nM=D\n')    # 重设被调用函数LCL地址
        self.file.write('@{0}\n0;JMP\n'.format(func_name))  # JUMP 被调用函数函数
        self.file.write('(RETURN_ADR{0})\n'.format(self.back_count))  # 返回标签设置
        self.back_count += 1

    def write_return(self):
        self.env.pop()
        self.file.write('@LCL\nD=M\n@5\nD=D-A\nA=D\nD=M\n@15\nM=D\n')  # temp中存返回地址
        self.file.write('@SP\nM=M-1\nA=M\nD=M\n@ARG\nM=D\n')  # 将返回结果存储到堆栈中,(返回后位置和返回前ARG基地址相同)
        self.file.write('@ARG\nD=M+1\n@SP\nM=D\n')  # 重设堆栈顶指针
        for mem, num in {'THAT': 1, 'THIS': 2, 'ARG': 3, 'LCL': 4}.items():
            self.file.write('@LCL\nD=M\n@{0}\nA=D-A\nD=M\n@{1}\nM=D\n'.format(num, mem))  # 恢复到原调用函数各种
        self.file.write('@15\nA=M\n0;JMP\n')  # 跳转返回地址

    def write_function(self, func_name, num):
        if func_name != 'Sys.init':
            self.env.append(func_name + '$')
        self.file.write('({0})\n'.format(func_name))
        for i in range(num):
            self.file.write('@SP\nA=M\nM=0\nD=A+1\n@SP\nM=D\n')  # 初始化LCL

    def close(self):
        self.file.close()


class VMTranslator(object):
    def __init__(self, file_or_dir):
        self.file_or_dir = file_or_dir
        self.file_writer = CodeWriter(file_or_dir[:-2] + 'asm' if '.vm' in file_or_dir else file_or_dir + '.asm')

    def run_one(self, file_uri):
        file_parser = Parser(file_uri)
        self.file_writer.set_filename(file_uri)
        while file_parser.has_more_commands():
            file_parser.advance()
            if file_parser.command_type == 'C_ARITHMETIC':
                self.file_writer.write_arithmetic(file_parser.arg_1)
            elif file_parser.command_type in ['C_PUSH', 'C_POP']:
                self.file_writer.write_push_pop(file_parser.command_type, file_parser.arg_1, file_parser.arg_2)
            elif file_parser.command_type == 'C_LABEL':
                self.file_writer.write_label(file_parser.arg_1)
            elif file_parser.command_type == 'C_GOTO':
                self.file_writer.write_goto(file_parser.arg_1)
            elif file_parser.command_type == 'C_IF':
                self.file_writer.write_if(file_parser.arg_1)
            elif file_parser.command_type == 'C_FUNCTION':
                self.file_writer.write_function(file_parser.arg_1, file_parser.arg_2)
            elif file_parser.command_type == 'C_RETURN':
                self.file_writer.write_return()
            elif file_parser.command_type == 'C_CALL':
                self.file_writer.write_call(file_parser.arg_1, file_parser.arg_2)

    def run_all(self):
        if os.path.isfile(self.file_or_dir):
            self.run_one(self.file_or_dir)
        else:
            files = [os.path.join(self.file_or_dir, f) for f in os.listdir(self.file_or_dir) if os.path.isfile(
                os.path.join(self.file_or_dir, f)) and f[-2:] == 'vm']
            for file_uri in files:
                self.run_one(file_uri)


if __name__ == '__main__':
    VMTranslator('/Users/alian/Desktop/nand2tetris/projects/08/FunctionCalls/StaticsTest').run_all()

```