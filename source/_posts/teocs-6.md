---
title: teocs-6
date: 2016-09-14 10:57:23
categories: programming
tags: course
---

### 从零开始构建现代计算机

#### 六.汇编编译器

##### tips

1.机器语言两重表现形式：汇编形式和二进制形式

2.符号的用途：①变量符号；②标签符号

3.符号解析：用户定义的变量名和符号标签与实际内存的映射通过符号表实现，

每遇到一个新的符号xxx，在符号表添加一行（xxx,n）n为分配给符号的地址

<!--more-->

项目：

```python
#!usr/bin/env python
# -*-coding:utf-8-*-
"""
author:wubaichuan

"""

import re
from sys import argv


class Parser(object):
    def __init__(self, file_uri):
        self.file = file_uri
        self.asm = self.__content()
        self.line = '//'

    def __content(self):
        with open(self.file, 'r') as f:
            temp = f.readlines()
        return temp

    def has_more_commands(self):
        return self.asm != []

    def advance(self):
        self.line = self.asm.pop(0).lstrip().strip()
        while not self.line or self.line[:2] == '//':
            self.line = self.asm.pop(0).lstrip()
        if '//' in self.line:
            self.line = self.line.split('//')[0].strip()

    def command_type(self):
        if self.line[0] == '@':
            return 'A_COMMAND'
        elif self.line[0] == '(':
            return 'L_COMMAND'
        else:
            return 'C_COMMAND'

    @property
    def symbol(self):
        if self.command_type() == 'A_COMMAND':
            m = re.match(r'@(\w+)', self.line)
            tmp = m.groups()[0]
        elif self.command_type() == 'L_COMMAND':
            tmp = self.line.lstrip('(').strip(')')
        else:
            raise ValueError
        return tmp

    @property
    def dest(self):
        if self.command_type() == 'C_COMMAND':
            if '=' in self.line:
                return self.line.split('=')[0]
            elif ';' in self.line:
                return 'null'
        raise ValueError

    @property
    def comp(self):
        if self.command_type() == 'C_COMMAND':
            if '=' in self.line:
                return self.line.split('=')[1]
            if ';' in self.line:
                return self.line.split(';')[0]
        raise ValueError

    @property
    def jump(self):
        if self.command_type() == 'C_COMMAND':
            if ';' in self.line:
                return self.line.split(';')[1]
            elif '=' in self.line:
                return 'null'
        raise ValueError


class Code(object):
    DEST_DICT = {'null': '000', 'M': '001', 'D': '010', 'MD': '011', 'A': '100', 'AM': '101', 'AD': '110', 'AMD': '111'}
    COMP_DICT = {'0': '0101010', '1': '0111111', '-1': '0111010', 'D': '0001100', 'A': '0110000',
                 '!D': '0001101', '!A': '0110001', '-D': '0001111', '-A': '0110011', 'D+1': '0011111',
                 'A+1': '0110111', 'D-1': '0001110', 'A-1': '0110010', 'D+A': '0000010', 'D-A': '0010011',
                 'A-D': '0000111', 'D&A': '0000000', 'D|A': '0010101', 'M': '1110000', '!M': '1110001',
                 '-M': '1110011', 'M+1': '1110111', 'M-1': '1110010', 'D+M': '1000010', 'D-M': '1010011',
                 'M-D': '1000111', 'D&M': '1000000', 'D|M': '1010101'}
    JUMP_DICT = {'null': '000', 'JGT': '001', 'JEQ': '010', 'JGE': '011', 'JLT': '100', 'JNE': '101',
                 'JLE': '110', 'JMP': '111'}

    @staticmethod
    def dest(signal):
        return Code.DEST_DICT[signal]

    @staticmethod
    def comp(signal):
        return Code.COMP_DICT[signal]

    @staticmethod
    def jump(signal):
        return Code.JUMP_DICT[signal]


class SymbolTable(object):
    SYMBOL_DICT = {}

    @staticmethod
    def constructor():
        SymbolTable.SYMBOL_DICT.update({'SP': 0, 'LCL': 1, 'ARG': 2, 'THIS': 3, 'THAT': 4,
                                        'R0': 0, 'R1': 1, 'R2': 2, 'R3': 3, 'R4': 4, 'R5': 5, 'R6': 6, 'R7': 7,
                                        'R8': 8, 'R9': 9, 'R10': 10, 'R11': 11, 'R12': 12, 'R13': 13, 'R14': 14,
                                        'R15': 15, 'SCREEN': 16384, 'KBD': 24576}
                                       )

    @staticmethod
    def add_entry(symbol, address):
        SymbolTable.SYMBOL_DICT[symbol] = address

    @staticmethod
    def contains(symbol):
        return symbol in SymbolTable.SYMBOL_DICT

    @staticmethod
    def get_address(symbol):
        return SymbolTable.SYMBOL_DICT[symbol]


def to_bin(num):
    result = 0
    n = 0
    num = int(num)
    while num:
        result += (num % 2) * 10 ** n
        n += 1
        num /= 2
    return result


def assembler(file_uri):
    """
    1.生成L指令中label的地址;
    2.生成hack
    
    """
    routine = Parser(file_uri)
    rom_address = 0
    temp_address = 16
    SymbolTable.constructor()
    while routine.has_more_commands():
        routine.advance()
        if routine.command_type() == 'L_COMMAND':
            temp_symbol = routine.symbol
            if not SymbolTable.contains(temp_symbol):
                SymbolTable.add_entry(temp_symbol, rom_address)
        else:
            rom_address += 1
    routine = Parser(file_uri)
    with open(file_uri.strip('asm') + 'hack', 'w') as f:
        while routine.has_more_commands():
            routine.advance()
            if routine.command_type() == 'A_COMMAND':
                temp_symbol = routine.symbol
                m = re.match(r'[^\d]', temp_symbol)
                if m:
                    if not SymbolTable.contains(temp_symbol):
                        SymbolTable.add_entry(temp_symbol, temp_address)
                        temp_address += 1
                    f.write("%016d\n" % to_bin(SymbolTable.get_address(temp_symbol)))
                else:
                    f.write("%016d\n" % to_bin(temp_symbol))

            elif routine.command_type() == 'C_COMMAND':
                f.write("111%s%s%s\n" % (Code.comp(routine.comp), Code.dest(routine.dest), Code.jump(routine.jump)))

if __name__ == '__main__':
    assembler('./pong/Pong.asm')
```