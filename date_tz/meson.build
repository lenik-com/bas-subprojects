project('date_tz', 'cpp', version: '1.0')

cpp = meson.get_compiler('cpp')

threads_dep = dependency('threads')

# Howard Hinnant date/time (libdate-tz)
# We vendor the needed headers and tz implementation to avoid relying on
# the target host having `libhowardhinnant-date-dev` installed.
date_tz_inc = include_directories('date/include')
date_tz_src = files('date/src/tz.cpp')
lib_date_tz_sys = cpp.find_library('date-tz', required: false)
date_exe_inc = []
if not lib_date_tz_sys.found()
    date_exe_inc = [date_tz_inc]
endif

# On POSIX hosts, prefer the OS zoneinfo database (/usr/share/zoneinfo) over the
# library default of ~/Downloads/tzdata (which breaks CI and machines without
# a manually downloaded IANA tzdata tree).
date_tz_cpp_args = []
if host_machine.system() != 'windows'
    date_tz_cpp_args += ['-DUSE_OS_TZDB=1']
endif

# Prefer the system lib if present; otherwise build/link our vendored tz.cpp.
# Placed after `add_project_arguments()` to satisfy Meson's ordering rules.
if lib_date_tz_sys.found()
    date_dep = declare_dependency(
        dependencies: [threads_dep, lib_date_tz_sys],
        compile_args: date_tz_cpp_args,
    )
else
    lib_date_tz = static_library(
        'date-tz',
        date_tz_src,
        include_directories: date_tz_inc,
        cpp_args: date_tz_cpp_args,
    )
    date_dep = declare_dependency(
        dependencies: [threads_dep],
        link_with: lib_date_tz,
        include_directories: date_tz_inc,
        compile_args: date_tz_cpp_args,
    )

    # ---- pkg-config ----
    pkgconfig = import('pkgconfig')
    pkgconfig.generate(
        lib_date_tz,
        name: 'date-tz',
        description: 'date-tz',
        version: meson.project_version(),
    )
endif

meson.override_dependency('date-tz', date_dep)
