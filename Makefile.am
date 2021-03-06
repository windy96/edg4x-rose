include $(top_srcdir)/config/Makefile.for.ROSE.includes.and.libs
include $(top_srcdir)/projects/compass/src/compassSupport/compass_dirs.inc


# DQ (1/18/2009): Added demo directory to support showoffs :-).
# DQ "config" and "src" must preceed any other directories, and 
# "tutorial" and "exampleTranslators" must preceed "docs" since 
# they generate data that is subsequently used in the generated 
# ROSE documentation (examples, input codes, and generated output).
# SUBDIRS = config src projects tests tutorial exampleTranslators docs developersScratchSpace tools proposals scripts
# SUBDIRS = libltdl config src projects tests tutorial exampleTranslators docs tools scripts demo binaries

# DQ (2/6/2010): changed the order of tests, the projects directory now follows the tests directory.
# This was something that was discussed in email previously with the group.  Thei point is to have the
# first tests run by make check be the most relevant tests (e.g. can we compile C++ code).
# SUBDIRS = libltdl config src tests projects tutorial exampleTranslators docs tools scripts demo binaries

# DQ (4/17/2010): Support for minimal configuration of ROSE (e.g. "--enable-only-fortran").
SUBDIRS = libltdl config src
if ROSE_BUILD_TESTS_DIRECTORY_SUPPORT
   SUBDIRS += tests
endif

if !ROSE_USE_CLANG_FRONTEND
if !ROSE_USE_INSURE
# DQ (4/23/2011): Ignoring all of these whole directories might be over kill.  But I have had problems with 
# Insure++ compilation specific to Hudson and want to start small and work to grow the number of parts of ROSE
# used by Insure.  Except on Hudson (e.g. on my machine) Insure++ can be used to compiler nearly all of ROSE.
if ROSE_BUILD_PROJECTS_DIRECTORY_SUPPORT
   SUBDIRS += projects
endif
if ROSE_BUILD_TUTORIAL_DIRECTORY_SUPPORT
 # If we can't process the tutorial directory then we can't generate example translators or the documentation
   SUBDIRS += tutorial exampleTranslators docs 
endif
SUBDIRS += tools scripts 
if ROSE_BUILD_TUTORIAL_DIRECTORY_SUPPORT
 # If we can't process the tutorial directory then we can't run demos
   SUBDIRS += demo
endif
SUBDIRS += binaries
# endif for controling Insure++ usage.
endif
endif

ACLOCAL_AMFLAGS = -I ./config -I ./acmacros -I ./libltdl -I /usr/share/aclocal

# Force rose.docs to be make by default 
# (if rose.docs.in is modified)
all-local: rose.docs 

all: enforce_policies remove_sage3basic_h_pch

core:
	$(MAKE) -C $(top_builddir)/src
	$(MAKE) -C $(top_builddir)/exampleTranslators
	$(MAKE) -C $(top_builddir)/tutorial

install-core: core rosePublicConfig.h
	$(MAKE) install-data-local
	$(MAKE) install -C $(top_builddir)/src
	$(MAKE) install -C $(top_builddir)/exampleTranslators
	$(MAKE) install -C $(top_builddir)/tutorial
	cp rosePublicConfig.h $(DESTDIR)$(includedir)/

install-compass: install-core
	$(MAKE) -C $(top_builddir)/projects/compass
	$(MAKE) install -C $(top_builddir)/projects/compass

# DQ (10/2/10): Of course this will over-rise the "make install" rule, so we can't 
# allow exactly this (what was I thinking?). Commented out until I get a better fix.
# DQ (9/29/2010): Next three lines are a modification suggested by Scott Warren to 
# support Eclipse.  Supports Eclipse feature called "makefile project with existing code" 
# which wants to have a single target to force the build.
#.PHONY: all-install

#all-install: all install

#install: all

remove_sage3basic_h_pch:
	touch $(top_builddir)/src/frontend/SageIII/sage3basic.h.pch
	rm    $(top_builddir)/src/frontend/SageIII/sage3basic.h.pch

# These are policies that we'd like to enforce for ROSE developers.  Things like:
#    1. source files should not use CR+LF line termination common on Windows platforms
#    2. header files should not include certain ROSE headers like "rose.h"
#    3. file names must be unique on a case-insensitive file system
#    4. header files must not have names that conflict with headers in other directories
#    5. etc.
# DO NOT DISABLE without first checking with a ROSE core developer
export prefix	# for use by policy scripts
enforce_policies:
	(cd $(top_srcdir) && ./scripts/policies-checker.sh)


# DQ (6/29/2004): I don't think we need this!
# LIBS = @LIBS@ ${SAGE_LIBS}

# It seems that it is better to handle the config directory with its own Makefile.am
# This allows us to hide autoconf accessory files (config.guess, etc.)
# Copy the config directory to the distribution: why?
# (because it has the file: config/Makefile.for.ROSE.includes.and.libs)
# EXTRA_DIST = stamp-h.in stamp-h1.in GNU_HEADERS config
# EXTRA_DIST = stamp-h.in stamp-h1.in rose.docs.in GNU_HEADERS COPYWRITE ROSE_ResearchPapers
# EXTRA_DIST = cmake CMakeLists.txt rose_config.h.in.cmake stamp-h.in stamp-h1.in rose.docs.in COPYRIGHT \
#              LicenseInformation ROSE_ResearchPapers README.OSX README.Cygwin build ChangeLog2 bincompat-sig
EXTRA_DIST = cmake CMakeLists.txt rose_config.h.in.cmake stamp-h.in stamp-h1.in rose.docs.in COPYRIGHT \
             LicenseInformation ROSE_ResearchPapers build ChangeLog2

# Liao, 12/11/2009. we should force this rule to run every time we want to make a distribution
# otherwise the stale one can go into the distribution package
cleanSig:
	rm -rf bincompat-sig
bincompat-sig: cleanSig
	$(srcdir)/scripts/bincompat-sig > $@

# At some point we want to put tals into the distribution (likely just PDF files, but not yet).
# ROSE_Talks

# clean-local explanation:
# Remove template repositories.  No source code is compiled, but configuration
#   tests may have created template repositories here.
clean-local:
	rm -rf Templates.DB ii_files ti_files *.csv

# Not sure if this should be part of the clean-local rule.
# chmod -R +rwx rose-0.9.5a; rm -rf rose-0.9.5a

# DQ (9/8/2006): Modified to remove backend specific header files
uninstall-local:
#	rm -rf $(DESTDIR)$(includedir)/*_HEADERS
	rm -rf `find $(DESTDIR)$(includedir)/ -iname "*_HEADERS"`

distclean-local:
	-rm -rf $(top_builddir)/include-staging/

# DQ (10/14/2010): We want to make sure that this does not go out to the install tree
# it also should not be in the distribution (since we want rose_config.h.in to be put 
# into the distribution instead; from which then rose_config.h is generated).
# However, the public version should be installed and distributed.
noinst_HEADERS = rose_config.h
include_HEADERS = rosePublicConfig.h

# DQ (4/5/2009): This rule is not called when compiling this directory, it is only
# called when compiling the /src/util directory (perhaps it should be located in 
# that directory's Makefile.am.
# DQ (12/3/2007): Added rose_paths.h so that it would be copied to the install tree.
# JJW (8/25/2008): Changed to a .c file
include $(top_srcdir)/config/build_rose_paths.Makefile

check-local: enforce_policies
	@echo '************************************************************'
	@echo '****** make check rule complete (terminated normally) ******'
	@echo '************************************************************'

install-exec-local:
	@echo '*****************************************************************'
	@echo '***** make install-exec rule complete (terminated normally) *****'
	@echo '*****************************************************************'

install-data-local: uninstall-local
	-mkdir $(DESTDIR)$(includedir)
#	DQ (11/1/2011): The EDG 4.x support does not require a separate include-staging directory.
#	if !ROSE_USE_NEW_EDG_INTERFACE
	cp -R $(top_builddir)/include-staging/* $(DESTDIR)$(includedir)/
#	endif
	@echo '*****************************************************************'
	@echo '***** make install-data rule complete (terminated normally) *****'
	@echo '*****************************************************************'

# DQ (4/22/2005): Set this so that make distcheck will use the same 
# --with-edg_source_code=true/false option as were used at configure.
# JJW (5/14/2008): Add the Boost flag the same way, using an internal
# variable from AX_BOOST_* to find the argument to --with-boost when
# this copy of ROSE was originally configured.
DISTCHECK_CONFIGURE_FLAGS = --with-boost=$(ac_boost_path)

# DQ (7/25/2008): If ROSE was originallly configured with Fortran (by tuning 
# on the java support) then make sure it is tested as part of the distcheck rule.
# if ROSE_USE_OPEN_FORTRAN_PARSER
if ROSE_USE_INTERNAL_JAVA_SUPPORT
# DISTCHECK_CONFIGURE_FLAGS += --with-java
  DISTCHECK_CONFIGURE_FLAGS += --with-java=$(JAVA_PATH)
else
# DQ (10/22/2010): Added specification of --without-java to distcheck rule.
# If we have first built without java then we have had to build some of the
# Fortran language supporting files as empty files and thus we don't want to
# run distcheck in a way that would use those files. However the problem
# when this is not used has more to do with the Java langauge support.
# A better reason why this is required might be so that we can run the NMI tests.
  DISTCHECK_CONFIGURE_FLAGS += --without-java
# TOO (3/24/2011): TODO: the new language-support configuration that I've
# implemented does not automatically disable fortran, whereas the previous
# implementation did. Since we now have Java-language support, the
# --without-java flag is confusing. I thought it meant "without java-language
# support". Speaking with Dan, we concluded that we need to look at renaming
# --without-java to something else like --without-jvm or --without-java-components
# This flag is suppose to turn off all parts of ROSE that use Java: 
# projects/javaports, java- and fortran-language support, etc.
  DISTCHECK_CONFIGURE_FLAGS += --disable-fortran
endif

# DQ (7/25/2008): If ROSE was originallly configured to use the HPC Toolkit
# then make sure it is tested as part of the distcheck rule.
if ROSE_BUILD_ROSEHPCT
  DISTCHECK_CONFIGURE_FLAGS += --enable-rosehpct
endif

# DQ (7/25/2008): If ROSE was originallly configured to use the javaport
# work (generation of Java interface functions for ROSE using SWIG) then 
# make sure it is tested as part of the distcheck rule.
if ENABLE_JAVAPORT
  DISTCHECK_CONFIGURE_FLAGS += --with-javaport
endif

# DQ (4/5/2010): Use the same version of OFP as specified on the original configure command line.
# CER (10/10/2011): No longer needed. A specific OFP version is configured by default and distributed with ROSE.
#DISTCHECK_CONFIGURE_FLAGS += --enable-ofp-version=@ROSE_OFP_MAJOR_VERSION_NUMBER@.@ROSE_OFP_MINOR_VERSION_NUMBER@.@ROSE_OFP_PATCH_VERSION_NUMBER@

# SRIRAM (8/2/2010): changing distcheck configuration for LLVM translator
if ROSE_USE_LLVM
   DISTCHECK_CONFIGURE_FLAGS += --with-llvm=$(llvm_path)
endif

# DQ (4/22/2005): To make clear when binaries are in the distribution as 
# opposed to EDG source code we rename the final generated distribution.
# JJW (8/3/2008): This is only used for source distributions now --
# source_with_binary_edg_dist does its own renaming
rename_distribution:
	new_name="$(distdir)-source-with-EDG-source.tar.gz"; \
	cp $(distdir).tar.gz $$new_name

# We reset distdir as shown on
# http://jcalcote.wordpress.com/2008/02/23/autoconf-macros-exposed-at3/ to add the SVN
# revision number
# The final distribution package should be
# rose-0.9.5a-8286.tar.gz
#
# Liao 12/14/2009
# We don't use subversion anymore
# We convert the number of git commits to a pseudo revision number instead
distdir = $(PACKAGE)-$(VERSION)$(DOT_SVNREV)

dist-hook:
	builddir="$$(pwd)" &&                                                      \
  (                                                                          \
  [ -e "$(srcdir)"/.git ] &&                                                 \
  cd $(srcdir) &&                                                            \
      git_head="$$(git rev-parse HEAD)" &&                                   \
      git_head_date="$$(git log -1 --format=%at $$(git rev-parse HEAD))" &&  \
          echo "$$git_head $$git_head_date" > $${builddir}/VERSION &&        \
          cp $${builddir}/VERSION $${builddir}/$(distdir)/VERSION            \
  ) ||                                                                       \
  (                                                                          \
  VERSION_FILE="$(srcdir)/VERSION" &&                                        \
  [ -e "$$VERSION_FILE" ] &&                                                 \
      cp "$$VERSION_FILE" $${builddir}/$(distdir)/VERSION                    \
  ) ||                                                                       \
  (                                                                          \
      echo "Error: could not determine SCM version information." &&          \
      exit 1                                                                 \
  ) ||                                                                       \
  exit 1

# Automake hook-ish rule to be run after dist rule (but hook runs before gzip so we can't use hook)
# We could build our own ROSE specific wrapper for dist and distcheck.
dist-after:
	@echo "Running dist hook rule ..."
	@$(MAKE) rename_distribution

# hook rule to be run after distcheck rule
distcheck-after:
	@echo "Running distcheck hook rule ..."
	@$(MAKE) rename_distribution

# CLEANFILES = rose_binary_compatibility_signature rose_binary_compatibility_signature_src
CLEANFILES = rose_binary_compatibility_signature bincompat-sig

find_rose_lock_files:
#	du -a | grep rose_performance_report_lockfile
	find . -name rose_performance_report_lockfile -print

# New rule to simplify generation of documentation (we want to avoid using the
# automake generated "docs" rule because the generation of documentation is
# dependent upon separate tools which the user might not have available
# (true for bothe the LaTeX and html documentation).
# DQ (7/23/2004): Since there is a "docs" subdirectory we need to force the build!
FORCE_DOCS_TO_BE_MADE:

# DQ (7/25/2008): Running "make docs" introduces dependences that are inappropriate
# for a distribution.
docs: FORCE_DOCS_TO_BE_MADE
	@if [ -f $(top_srcdir)/src/midend/midend.docs ]; then \
	   echo "    Generate all possible documentation cd docs; make documentation;"; \
	   $(MAKE) -Ctutorial docs && \
	   $(MAKE) -Cprojects/haskellport docs && \
	   $(MAKE) -Ctests/roseTests/astInterfaceTests check && \
	   $(MAKE) -Ctests/roseTests/ompLoweringTests check && \
	   $(MAKE) -Ctests/CompileTests/UPC_tests check && \
	   $(MAKE) -Cprojects/UpcTranslation/tests check && \
	   $(MAKE) -Cprojects/autoParallelization/tests check && \
	   $(MAKE) -Cdocs docs && \
	   $(MAKE) -C$(compass_build_tooldir)/compass docs; \
	else \
	   echo "Please look at the ROSE web site for documentation (www.roseCompiler.org)"; \
	fi

if ROSE_HAS_EDG_SOURCE

# 1
# This rule generates the simple binary tarball for the directories: EDG and EDG_SAGE_Connection
binary_edg_tarball: 
	$(MAKE) -C src/frontend/CxxFrontend binary_edg_tarball

upload_edg_binary:
	$(MAKE) -C src/frontend/CxxFrontend upload_edg_binary

endif

# Makefile rule to test the Hudson runTest script (used for all Hudson tests).
# Note that empty ROSE_TEST_BUILD_STYLE maps to a minimal build.
testHudsonRunScript:
	export ROSE_TEST_BUILD_SKIP_BUILD_SCRIPT=yes; \
   export ROSE_TEST_GCC_VERSION=4.2.4; \
   export ROSE_TEST_BUILD_STYLE=; \
   export ROSE_TEST_BOOST_PATH=${MY_BOOST_ROOT}; \
   export ROSE_TEST_JAVA_PATH=${JAVA_PATH}; \
   cd $(top_srcdir); \
   $(top_srcdir)/scripts/hudson/runTest

# This will cause the Hudson environment to be dumped to a file so that
# it can be sourced to reproduce the environment required to run tests
# in the hudson tmp directories as part of debugging.
dumpHudsonEnv:
	export ROSE_DUMP_HUDSON_ENV=yes; $(MAKE) testHudsonRunScript


# $(top_srcdir)/scripts/hudson/testHudsonRunScript.sh;


# DQ (12/16/2009): Added interesting "git" feature by Liao.
# It is unclear if it is very meaningful since it represents 
# individual work style more than anything else (but it's fun).
CheckinCounts:
	@echo "*****************************************************"
	@echo "Generated list of checkins to ROSE sumarized by user."
	@echo "*****************************************************"
	cd $(top_srcdir); git shortlog -s -n
	@echo "*****************************************************"


# This rule is useful for identifing source files where "rose_config.h" should be included.
FindFileRequiringRoseConfigHeaderShort:
#	grep "#define" rose_config.h | cut --delimiter=" " -f 2 
#	@list=`grep \"\#define\" rose_config.h | cut --delimiter=" " -f 2`; for f in $$list; do 
#	list=`ls -l | cut --delimiter=" " -f 2`; for f in $$list; do
	list=`egrep "#define|\/* #undef" rose_config.h | sed 's/\/\* //g' | egrep -v "const|error_t|inline|off_t|size_t" | cut --delimiter=" " -f 2`; \
   for f in $$list; do \
        echo "Searching for -->$$f<-- macro"; \
        grep -r $$f $(top_srcdir)/src/* | grep "\.h:" | cut --delimiter=":" -f 1; \
   done

# This rule is useful for identifing header files where "rose_config.h" is required and 
# thus should be fixed to not require this file. In the case of #undef it is represented 
# as "/* #undef" so we have to detect and remove the "/* " using sed.
# And ignore the macro definitions for "const|error_t|inline|off_t|size_t".
FindFileRequiringRoseConfigHeaderLong:
	list=`egrep "#define|\/* #undef" rose_config.h | sed 's/\/\* //g' | egrep -v "const|error_t|inline|off_t|size_t" | cut --delimiter=" " -f 2`; \
   for f in $$list; do \
        echo "Searching for -->$$f<-- macro"; \
        grep -r $$f $(top_srcdir)/* | grep "\.h:"; \
   done

FindFileRequiringRoseConfigSourceLong:
	list=`egrep "#define|\/* #undef" rose_config.h | sed 's/\/\* //g' | egrep -v "const|error_t|inline|off_t|size_t" | cut --delimiter=" " -f 2`; \
   for f in $$list; do \
        echo "Searching for -->$$f<-- macro"; \
        grep -r $$f $(top_srcdir)/* | grep "\.C:"; \
   done

FindFileRequiringRoseConfigSourceAndHeadersLong:
	list=`egrep "#define|\/* #undef" rose_config.h | sed 's/\/\* //g' | egrep -v "const|error_t|inline|off_t|size_t" | cut --delimiter=" " -f 2`; \
   for f in $$list; do \
        echo "Searching for -->$$f<-- macro"; \
        grep -r $$f $(top_srcdir)/src/* | egrep "\.C:|\.h:"; \
   done

FILTER_MAKEFILES = grep "Makefile\.am" | grep -v "Makefile\.am~"
NumberOfMakefiles:
	@echo "Number of Makefile.am files in root directory: "
	@du -a $(srcdir) | $(FILTER_MAKEFILES) | wc -l
	@echo "Number of Makefile.am files in src directory: "
	@du -a $(srcdir)/src | $(FILTER_MAKEFILES) | wc -l
	@echo "Number of Makefile.am files in projects directory: "
	@du -a $(srcdir)/projects | $(FILTER_MAKEFILES) | wc -l
	@echo "Number of Makefile.am files in tests directory: "
	@du -a $(srcdir)/tests | $(FILTER_MAKEFILES) | wc -l
	@echo "Number of Makefile.am files in exampleTranslators directory: "
	@du -a $(srcdir)/exampleTranslators | $(FILTER_MAKEFILES) | wc -l
	@echo "Number of Makefile.am files in tutorial directory: "
	@du -a $(srcdir)/tutorial | $(FILTER_MAKEFILES)| wc -l
	@echo "Number of Makefile.am files in docs directory: "
	@du -a $(srcdir)/docs | $(FILTER_MAKEFILES) | wc -l
	@echo "Number of Makefile.am files in demo directory: "
	@du -a $(srcdir)/demo | $(FILTER_MAKEFILES) | wc -l
	@echo "Number of Makefile.am files in binaries directory: "
	@du -a $(srcdir)/binaries | $(FILTER_MAKEFILES) | wc -l
	@echo "Number of Makefile.am files in tools directory: "
	@du -a $(srcdir)/tools | $(FILTER_MAKEFILES) | wc -l
	@echo "Number of Makefile.am files in config directory: "
	@du -a $(srcdir)/config | $(FILTER_MAKEFILES) | wc -l

# ***********************************************************
#      Display Which Automake Conditionals Are Defined
# ***********************************************************
# DQ (10/18/2010): Added test for names used in automake conditional macros
# (eventually we want tests for consistancy).  This rule generates a makefile
# containing a makefile rule. The makefile rule can be run to see what automake
# conditionals were defined in the last run of configure. It might be that 
# this mechanism should be in a script generated by the build script, but we 
# need an initial version of the generated script to be checked into git so that 
# the build script can find it.  Note that we grep the Makefile.in instead of
# the Makefile.am (which would be more difficult, I think). So we need the
# Makefile.in from runing the build script first.  Thus having the generated
# file checked into git avoids a dependence on a file that we would not have.
FILTER_AUTOMAKE_CONDITONALS = egrep -v "autom4te" | grep -v "~" | grep AM_CONDITIONAL
TEST_FILE = $(top_srcdir)/config/automake_conditional_display_makefile
NameOfAutomakeConditionals:
#	grep -r "^AM_CONDITIONAL" $(top_srcdir) | $(FILTER_AUTOMAKE_CONDITONALS)
#	grep "_TRUE@" $(top_srcdir)/Makefile.in | grep "@" | cut --delimiter="@" -f 2 | sed s/_TRUE//g | sort --unique | grep -v "top_srcdir"
	list=`grep "_TRUE@" $(top_srcdir)/Makefile.in | grep "@" | cut --delimiter="@" -f 2 | sed 's/_TRUE//g' | sort --unique | grep -v "top_srcdir"`; \
   rm -f $(TEST_FILE); \
   touch $(TEST_FILE); \
   echo "automake_conditional_display:" >> $(TEST_FILE); \
   for f in $$list; do \
        echo "Processing -->$$f<-- macro"; \
        echo "if $$f" >> $(TEST_FILE); \
        echo -e "\t@echo \"Automake conditional $$f: DEFINED\"" >> $(TEST_FILE); \
        echo "else" >> $(TEST_FILE); \
        echo -e "\t@echo \"Automake conditional $$f: NOT defined\"" >> $(TEST_FILE); \
        echo "endif" >> $(TEST_FILE); \
   done
#	This can't be run since it can't be processed through automake and autoconf.
#	$(MAKE) -f $(TEST_FILE) automake_conditional_display

# This is a test of the new make rule added in the file $(TEST_FILE).
testNameOfAutomakeConditionals:
	@echo "*************************************************************************"
	@echo "Testing makefile rule automake_conditional_display in file: $(TEST_FILE)."
	@echo "*************************************************************************"
	$(MAKE) automake_conditional_display

# Include this file (generated first by the rule down below and checked into our git repository).
# This could be generated by our build script.
include $(top_srcdir)/config/automake_conditional_display_makefile
# ***********************************************************



# DQ (10/18/2010): These are used all over within a line (so don't use start of line anchor).
# (eventually we want tests for consistancy)
FILTER_AUTOCONF_DEFINE = egrep -v "autom4te" | grep -v "~" | grep -v "configure:" | grep AC_DEFINE
NameOfAutoconfDefines:
	grep -r "AC_DEFINE" $(top_srcdir) | $(FILTER_AUTOCONF_DEFINE)




# DQ (2/12/2010): Report the areas of ROSE where we have skipped some tests of ROSE 
# either doing analysis, code generation, or translation of the ROSE source code.
ProblemAreas:
	echo "Directories where ROSE does not analyze code for some ROSE source files."
	grep -r ROSE_USING_ROSE_ANALYSIS $(top_srcdir) | grep Makefile.am
	echo "Directories where ROSE does not generate code for some ROSE source files."
	grep -r ROSE_USING_ROSE_CODE_GENERATION $(top_srcdir) | grep Makefile.am
	echo "Directories where ROSE does not translate (compile) code for some ROSE source files."
	grep -r ROSE_USING_ROSE_TRANSLATE $(top_srcdir) | grep Makefile.am
	echo "Directories where ROSE does not properly handled AST File I/O for at least some ROSE source files."
	grep -r ROSE_USING_ROSE_AST_FILE_IO $(top_srcdir) | grep Makefile.am
	echo "Files where ROSE does not analyze some amount of code (test of USE_ROSE in each ROSE source file)."
	grep -rl USE_ROSE $(top_srcdir) | grep "\.C"
	echo "Files where ROSE does not analyze some amount of code (test of CXX_IS_ROSE_ANALYSIS in each ROSE source file)."
	grep -rl CXX_IS_ROSE_ANALYSIS $(top_srcdir) | grep "\.C"
	echo "Files where ROSE does not analyze some amount of code (test of CXX_IS_ROSE_CODE_GENERATION"
	echo "in each ROSE source file)."
	grep -rl CXX_IS_ROSE_CODE_GENERATION $(top_srcdir) | grep "\.C"
#	echo "Files where ROSE does not analyze some amount of code (test of CXX_IS_ROSE_TRANSLATOR in each ROSE source file)."
#	grep -rl CXX_IS_ROSE_TRANSLATOR $(top_srcdir) | grep "\.C"



# Report the version number of the ROSE translator built.
PrintRoseTranslatorVersion: $(top_builddir)/tests/testAnalysis
	$(top_builddir)/tests/testAnalysis --version

release_binary_compatibility_signature=$(shell ${top_srcdir}/scripts/bincompat-sig)

PrintBinaryCompatibilitySignature:
	@echo "release_binary_compatibility_signature = ${release_binary_compatibility_signature}"

# Generate public version of rose_config.h, which is triggered by the AC_CONFIG_COMMANDS macro near the end
# of config/support-rose.m4 whenever config.status runs.
rosePublicConfig.h: rose_config.h
	@echo "  GEN     $@"
	@$(top_srcdir)/scripts/publicConfiguration.pl <$^ >$@
DISTCLEANFILES = rosePublicConfig.h

# This file does not exist until we run the compiler to build it.
# include $(top_builddir)/src/frontend/SageIII/astPostProcessing/.libs/astPostProcessing.d

