#
# use make run-test PROFILE=net_2_0
#

thisdir = tests
SUBDIRS =
include ../build/rules.make

DISTFILES = README.tests $(wildcard dlls/**/*.cs)
DISTFILES += $(wildcard *.cs) $(wildcard *.il) $(wildcard *.xml) $(wildcard *.inc) $(wildcard known-issues-*) $(wildcard *.snk)

with_mono_path = MONO_PATH="$(topdir)/class/lib/$(PROFILE)$(PLATFORM_PATH_SEPARATOR)$$MONO_PATH"

ilasm = $(topdir)/class/lib/$(PROFILE)/ilasm.exe
ILASM = MONO_PATH="$(topdir)/class/lib/$(PROFILE)$(PLATFORM_PATH_SEPARATOR)$$MONO_PATH" $(RUNTIME) $(RUNTIME_FLAGS) $(ilasm)

USE_MCS_FLAGS :=

# mention all targets
all-local $(STD_TARGETS:=-local):

VALID_PROFILE := $(filter net_2_0 moonlight net_4_0, $(PROFILE))
ifdef VALID_PROFILE
# casts
bootstrap-cast.exe: gen-cast-test.cs
	$(BOOT_COMPILE) -target:exe /out:$@ $<

casts.cs: bootstrap-cast.exe
	$(with_mono_path) $(RUNTIME) $(RUNTIME_FLAGS) $< >$@

casts-mcs.exe: casts.cs
	$(CSCOMPILE) -target:exe /out:$@ $<

casts-boot.exe: casts.cs
	$(BOOT_COMPILE) -target:exe /out:$@ $<

boot-casts.out: casts-boot.exe
	$(with_mono_path) $(RUNTIME) $(RUNTIME_FLAGS) $< >$@

mcs-casts.out: casts-mcs.exe
	$(with_mono_path) $(RUNTIME) $(RUNTIME_FLAGS) $< >$@

test-casts: boot-casts.out mcs-casts.out
	cmp $^
	-rm -f bootstrap-cast.exe casts.cs casts-boot.exe casts-mcs.exe boot-casts.out mcs-casts.out

ifeq (net_4_0, $(PROFILE))
COMPILER_NAME = dmcs
TEST_PATTERN = 'v4'
LOCAL_RUNTIME_FLAGS = --verify-all
DEFINES = -compiler-options:"-d:NET_4_0"
#TOPTIONS += '-il:ver-il-dmcs.xml'
endif
ifeq (moonlight, $(PROFILE))
COMPILER_NAME = smcs
TEST_PATTERN = 'v2'
LOCAL_RUNTIME_FLAGS = --security=temporary-smcs-hack
DEFINES = -compiler-options:"-d:MOONLIGHT"
endif
ifeq (net_2_0, $(PROFILE))
COMPILER_NAME = gmcs
TEST_PATTERN = 'v2'
LOCAL_RUNTIME_FLAGS = --verify-all
TOPTIONS += '-il:ver-il-gmcs.xml'
endif

COMPILER = $(topdir)/class/lib/$(PROFILE)/$(COMPILER_NAME).exe
TESTER = MONO_RUNTIME='$(RUNTIME)' $(TEST_RUNTIME) $(RUNTIME_FLAGS) $(LOCAL_RUNTIME_FLAGS) $(topdir)/class/lib/$(PROFILE)/compiler-tester.exe

TEST_ILS := $(wildcard *-lib.il)

build-compiler-lib:
	cd ../class/Mono.CSharp && $(MAKE) NO_DIR_CHECK=yes

qcheck: build-compiler-lib
	$(TESTER) -mode:pos -files:$(TEST_PATTERN) -compiler:$(COMPILER) -issues:known-issues-$(COMPILER_NAME) -log:$(COMPILER_NAME).log $(TOPTIONS) $(DEFINES)

test-local:
	@:

run-test-local: $(TEST_ILS:.il=.dll) setup qcheck

endif

clean-local:
	-rm -fr dir-*
	-rm -f *.exe *.dll *.netmodule *.out *.pdb *.mdb casts.cs *.log
	-rm -f xml-*.xml

dist-local: dist-default
	rm -f $(distdir)/casts.cs

csproj-local:

%-il.dll: %-il.il
	$(ILASM) /dll $<

%-lib.dll: %-lib.il
	$(ILASM) /dll /out:$@ $<

setup:
	$(ILASM) /dll property-il.il
	$(CSCOMPILE) /r:property-il.dll property-main.cs /out:property-main.exe
	$(TEST_RUNTIME) property-main.exe

	$(CSCOMPILE) -t:library dlls/test-679-2/test-679-lib-2.cs
	$(CSCOMPILE) -t:library dlls/test-679-1/test-679-lib.cs -r:dlls/test-679-2/test-679-lib-2.dll
