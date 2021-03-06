# Copyright (c) 2001-2004, Roger Dingledine
# Copyright (c) 2004-2006, Roger Dingledine, Nick Mathewson
# Copyright (c) 2007-2015, The Tor Project, Inc.
# See LICENSE for licensing information

# "foreign" means we don't follow GNU package layout standards
# 1.9 means we require automake vesion 1.9
AUTOMAKE_OPTIONS = foreign 1.9 subdir-objects

ACLOCAL_AMFLAGS = -I m4

noinst_LIBRARIES=
EXTRA_DIST=
noinst_HEADERS=
bin_PROGRAMS=
CLEANFILES=
TESTS=
noinst_PROGRAMS=
DISTCLEANFILES=
bin_SCRIPTS=
AM_CPPFLAGS=
AM_CFLAGS = @TOR_SYSTEMD_CFLAGS@
include src/include.am
include doc/include.am
include contrib/include.am

EXTRA_DIST+= \
	ChangeLog					\
	INSTALL						\
	LICENSE						\
	Makefile.nmake					\
	README						\
	ReleaseNotes

if COVERAGE_ENABLED
TEST_CFLAGS=-fno-inline -fprofile-arcs -ftest-coverage
else
TEST_CFLAGS=
endif

#install-data-local:
#	$(INSTALL) -m 755 -d $(LOCALSTATEDIR)/lib/tor

# Allows to override rpmbuild with rpmbuild-md5 from fedora-packager so that
# building for EL5 won't fail on https://bugzilla.redhat.com/show_bug.cgi?id=490613
RPMBUILD ?= rpmbuild

# Use automake's dist-gzip target to build the tarball
dist-rpm: dist-gzip
	TIMESTAMP=$$(date +"%Y-%m-%d_%H.%M.%S"); 				\
	RPM_BUILD_DIR=$$(mktemp -d "/tmp/tor-rpm-build-$$TIMESTAMP-XXXX");	\
	mkdir -p "$$RPM_BUILD_DIR"/{BUILD,RPMS,SOURCES/"tor-$(VERSION)",SPECS,SRPMS}; \
	cp -fa "$(distdir).tar.gz" "$$RPM_BUILD_DIR"/SOURCES/;			\
	LIBS=-lrt $(RPMBUILD) -ba --define "_topdir $$RPM_BUILD_DIR" tor.spec; 	\
	cp -fa "$$RPM_BUILD_DIR"/SRPMS/* .;                           		\
	cp -fa "$$RPM_BUILD_DIR"/RPMS/* .;                            		\
	rm -rf "$$RPM_BUILD_DIR";						\
	echo "RPM build finished";						\
	#end of dist-rpm

dist: check

doxygen:
	doxygen && cd doc/doxygen/latex && make

test: all
	./src/test/test

# Note that test-network requires a copy of Chutney in $CHUTNEY_PATH.
# Chutney can be cloned from https://git.torproject.org/chutney.git .
test-network: all
	./src/test/test-network.sh

test-stem: $(TESTING_TOR_BINARY)
	@if test -d "$$STEM_SOURCE_DIR"; then \
		"$$STEM_SOURCE_DIR"/run_tests.py --tor $(TESTING_TOR_BINARY) --all --log notice --target RUN_ALL; \
	else \
		echo '$$STEM_SOURCE_DIR was not set.'; echo; \
		echo "To run these tests, git clone https://git.torproject.org/stem.git/ ; export STEM_SOURCE_DIR=\`pwd\`/stem"; \
	fi


reset-gcov:
	rm -f src/*/*.gcda src/*/*/*.gcda

HTML_COVER_DIR=./coverage_html
coverage-html: all
	test -e "`which lcov`" || (echo "lcov must be installed. See <http://ltp.sourceforge.net/coverage/lcov.php>." && false)
	test -d "$(HTML_COVER_DIR)" || mkdir -p "$(HTML_COVER_DIR)"
	lcov --rc lcov_branch_coverage=1 --directory ./src --zerocounters
	$(MAKE) reset-gcov
	$(MAKE) check
	lcov --capture --rc lcov_branch_coverage=1 --no-external --directory . --output-file "$(HTML_COVER_DIR)/lcov.tmp"
	lcov --remove "$(HTML_COVER_DIR)/lcov.tmp" --rc lcov_branch_coverage=1 'test/*' 'ext/tinytest*' '/usr/*' --output-file "$(HTML_COVER_DIR)/lcov.info"
	genhtml --branch-coverage -o "$(HTML_COVER_DIR)" "$(HTML_COVER_DIR)/lcov.info"

# Avoid strlcpy.c, strlcat.c, aes.c, OpenBSD_malloc_Linux.c, sha256.c,
# eventdns.[hc], tinytest*.[ch]
check-spaces:
	./scripts/maint/checkSpace.pl -C              \
		src/common/*.[ch]		      \
		src/or/*.[ch]			      \
		src/test/*.[ch]			      \
		src/tools/*.[ch]		      \
		src/tools/tor-fw-helper/*.[ch]

check-docs:
	./scripts/maint/checkOptionDocs.pl

check-logs:
	./scripts/maint/checkLogs.pl                  \
		src/*/*.[ch] | sort -n

version:
	@echo "Tor @VERSION@"
	@if test -d "$(top_srcdir)/.git" && test -x "`which git 2>&1;true`"; then \
	   echo -n "git: " ;\
	   (cd "$(top_srcdir)" && git rev-parse --short=16 HEAD); \
	fi

mostlyclean-local:
	rm -f src/*/*.gc{da,no} src/*/*/*.gc{da,no}
