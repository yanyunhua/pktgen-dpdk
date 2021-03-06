# SPDX-License-Identifier: BSD-3-Clause
# Copyright(c) 2019 Intel Corporation

libraries = ['common', 'utils', 'vec', 'plugin', 'cli', 'lua']

default_cflags = machine_args
if cc.has_argument('-Wno-format-truncation')
	default_cflags += '-Wno-format-truncation'
endif

# Set default cflags
default_cflags += '-D_GNU_SOURCE'

enabled_libs = []
foreach l:libraries
	build = true
	name = l
	version = 1
	allow_experimental_apis = false
	sources = []
	headers = []
	includes = []
	cflags = default_cflags
	objs = [] # other object files to link against, used e.g. for
	          # instruction-set optimized versions of code

	# use "deps" for internal Pktgen dependencies, and "ext_deps" for
	# external package/library requirements
	ext_deps = []
	deps = []

	dir_name = l
	message('Process ' + dir_name)
	subdir(dir_name)

	if build
		enabled_libs += name
		install_headers(headers)

		libname = name
		includes += include_directories(dir_name)
		includes += dpdk_incs

		if sources.length() == 0
			message('    No sources in '+ libname)
			# if no C files, just set a dependency on header path
			shared_dep = declare_dependency(include_directories: includes)
			static_dep = shared_dep
		else
			message('    Sources in '+ libname)
			shared_deps = ext_deps
			static_deps = ext_deps
			foreach d:deps
				if not is_variable('static_pktgen_' + d)
					error('Missing dependency ' + d +
						' for library ' + libname)
				else
					message('    === Found dependency ' + d)
				endif
				shared_deps += [get_variable('shared_pktgen_' + d)]
				static_deps += [get_variable('static_pktgen_' + d)]
			endforeach

			if allow_experimental_apis
				cflags += '-DALLOW_EXPERIMENTAL_API'
			endif

			lib_version = major_version
			so_version = major_version

			# first build static lib
			static_lib = static_library(libname,
					sources,
					objects: objs,
					c_args: cflags,
					dependencies: static_deps,
					include_directories: includes,
					install: true)
			static_dep = declare_dependency(link_with: static_lib,
					include_directories: includes,
					dependencies: static_deps)

			# then use pre-build objects to build shared lib
			sources = []
			objs += static_lib.extract_all_objects(recursive: false)
			shared_lib = shared_library(libname,
					sources,
					objects: objs,
					c_args: cflags,
					dependencies: shared_deps,
					include_directories: includes,
					version: lib_version,
					soversion: so_version,
					install: true)
			shared_dep = declare_dependency(link_with: shared_lib,
					include_directories: includes,
					dependencies: shared_deps)

			pktgen_shared_libraries = [shared_lib] + pktgen_shared_libraries
			pktgen_static_libraries = [static_lib] + pktgen_static_libraries
		endif # sources.length() > 0

		set_variable('shared_pktgen_' + libname, shared_dep)
		set_variable('static_pktgen_' + libname, static_dep)
	endif # if build
endforeach
