PG_CONFIG = pg_config
MIN_SUPPORTED_VERSION = $(shell cat src/init.c | grep -m 1 'MIN_SUPPORTED_VERSION_STR' | sed "s/^\#define MIN_SUPPORTED_VERSION_STR \"\([0-9]*\.[0-9]*\)\"$\/\1/g")

# will need to change when we support more than one version
ifneq ($(MIN_SUPPORTED_VERSION), $(shell $(PG_CONFIG) --version | sed "s/^PostgreSQL \([0-9]*\.[0-9]*\)\.[0-9]*$\/\1/g"))
$(error "TimescaleDB requires PostgreSQL $(MIN_SUPPORTED_VERSION)")
endif

EXT_GIT_COMMIT := $(shell git describe --abbrev=4 --dirty --always --tags || echo $(EXT_GIT_COMMIT))

MODULE_big = timescaledb

SRCS = \
	src/init.c \
	src/load.c \
	src/extension.c

OBJS = $(SRCS:.c=.o)
DEPS = $(SRCS:.c=.d)

PGXS := $(shell $(PG_CONFIG) --pgxs)

include $(PGXS)
override CFLAGS += -DINCLUDE_PACKAGE_SUPPORT=0 -MMD -DEXT_GIT_COMMIT=\"$(EXT_GIT_COMMIT)\"
-include $(DEPS)

clean: clean-extra

clean-extra:
	@rm -f src/*~
	@rm -f src/*.d
	@rm -f src/*.o

package: clean
	@mkdir -p package/lib
	@mkdir -p package/extension
	$(install_sh) -m 755 $(EXTENSION).so 'package/lib/$(EXTENSION).so'
	$(install_sh) -m 644 $(EXTENSION).control 'package/extension/'

typedef.list: clean $(OBJS)
	../scripts/generate_typedef.sh

pgindent: typedef.list
	pgindent --typedef=typedef.list

.PHONY: check-sql-files all
