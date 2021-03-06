# The MIT License (MIT)
#
# Copyright (c) 2018 Steven Michaud
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in all
# copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
# SOFTWARE.

# Alternate Makefile that allows us to use low-level llvm tools to build
# step-by-step.  One of the steps generates LLVM intermediate language
# files (ending in *.ii), which support calling conventions that aren't
# supported in C/C++ files, but which sometimes get used by clang for internal
# (non-public) functions -- for example fastcc or coldcc.
#
# http://llvm.org/docs/LangRef.html#calling-conventions
# https://clang.llvm.org/docs/AttributeReference.html#calling-conventions
#
# To make a C/C++ function in your hook library use the fastcc calling
# convention, label it with __attribute__((fastcall)) in C/C++ code and use
# this makefile to build the hook library.  Under the hook-i386.ii target we
# use sed to replace all instances of "x86_fastcallcc" (the internal name of
# the fastcall calling convention) with "fastcc".
#
# Internal calling conventions are deliberately non-standardized.  So it's
# important, when building code that hooks methods which use them, to use
# tools that are as compatible as possible with the code that you're hooking.
# I've had good luck up through Sierra with the LLVM 3.9.0 Clang download:
# http://releases.llvm.org/3.9.0/clang+llvm-3.9.0-x86_64-apple-darwin.tar.xz.
# HighSierra seems to require the LLVM 4.0.0 Clang download:
# http://releases.llvm.org/4.0.0/clang+llvm-4.0.0-x86_64-apple-darwin.tar.xz.

#LLVM_HOME=/usr/local/clang+llvm-4.0.0-x86_64-apple-darwin
LLVM_HOME=/usr/local/clang+llvm-3.9.0-x86_64-apple-darwin
CLANG=$(LLVM_HOME)/bin/clang++
LLAS=$(LLVM_HOME)/bin/llvm-as
LLC=$(LLVM_HOME)/bin/llc
LIPO=lipo
SED=sed

hook.dylib : hook.o
	$(CLANG) -arch i386 -arch x86_64 -o hook.dylib hook.o \
		-Wl,-read_only_relocs,suppress \
		-lobjc -framework Cocoa -framework Carbon \
		-Wl,-F/System/Library/PrivateFrameworks -framework CoreSymbolication \
		-dynamiclib

hook.o : hook-i386.o hook-x86_64.o
	$(LIPO) -create -arch i386 hook-i386.o -arch x86_64 hook-x86_64.o \
		-output hook.o

hook-x86_64.o : hook-x86_64.s
	$(CLANG) -arch x86_64 -o hook-x86_64.o -c hook-x86_64.s

hook-i386.o : hook-i386.s
	$(CLANG) -arch i386 -o hook-i386.o -c hook-i386.s

hook-x86_64.s : hook-x86_64.bc
	$(LLC) hook-x86_64.bc -o hook-x86_64.s

hook-i386.s : hook-i386.bc
	$(LLC) hook-i386.bc -o hook-i386.s

hook-x86_64.bc : hook-x86_64.ii
	$(LLAS) -o=hook-x86_64.bc hook-x86_64.ii

hook-i386.bc : hook-i386.ii
	$(LLAS) -o=hook-i386.bc hook-i386.ii

hook-x86_64.ii : hook.mm
	$(CLANG) -arch x86_64 -S -emit-llvm -o hook-x86_64.ii \
		-Wno-deprecated-declarations -c hook.mm

hook-i386.ii : hook.mm
	$(CLANG) -arch i386 -S -emit-llvm -o hook-i386.iii \
		-Wno-deprecated-declarations -c hook.mm
	$(SED) -e 's/ x86_fastcallcc / fastcc /' hook-i386.iii > hook-i386.ii

clean :
	rm hook.o hook.dylib hook-i386* hook-x86_64*
