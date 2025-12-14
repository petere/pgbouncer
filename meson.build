project('PgBouncer',
        ['c'],
        version: '1.24.1',
        license: 'ISC',
        meson_version: '>=0.58',
        default_options: [
          'buildtype=debugoptimized', # -O2 + debug
        ],
)

fs = import('fs')

cc = meson.get_compiler('c')

not_found_dep = dependency('', required: false)

cdata = configuration_data()


use_evdns = get_option('evdns')

libevent = dependency('libevent', required: true)

#
# cares option
#
libcares = not_found_dep
caresopt = get_option('cares')
libcares = dependency('libcares', required: caresopt, version: '>= 1.6.0')
cdata.set('USE_CARES', libcares.found() ? 1 : false,
          description: 'Use c-ares for name resolution.')
if libcares.found()
  use_evdns = false
endif

#
# pam option
#
pam = not_found_dep
pamopt = get_option('pam')
if not pamopt.disabled()
  pam = dependency('pam', required: false)

  if not pam.found()
    pam = cc.find_library('pam', required: pamopt)
  endif

  if pam.found()
    cdata.set('HAVE_PAM', 1,
              description: 'PAM support')
  endif
endif

#
# ldap option
#
libldap = not_found_dep
ldapopt = get_option('ldap')
libldap = dependency('ldap', required: ldapopt)
cdata.set('HAVE_LDAP', libldap.found() ? 1 : false,
          description: 'Build with LDAP support.')

threads_dep = dependency('threads',
                        required: ldapopt.enabled() or pamopt.enabled())

#
# openssl option
#
openssl = not_found_dep
opensslopt = get_option('openssl')
openssl = dependency('openssl', required: opensslopt)
cdata.set('USUAL_LIBSSL_FOR_TLS', openssl.found() ? 1 : false,
          description: 'Use libssl for TLS.')
cdata.set('OPENSSL_API_COMPAT', '0x00908000L',
          description: 'Define to the OpenSSL API version in use. This avoids deprecation warnings from newer OpenSSL versions.')

cdata.set_quoted('USUAL_TLS_CA_FILE', get_option('root-ca-file'))

#
# systemd option
#
systemd = not_found_dep
systemdopt = get_option('systemd').disable_auto_if(host_machine.system() != 'linux')
systemd = dependency('libsystemd', required: systemdopt)
cdata.set('USE_SYSTEMD', systemd.found() ? 1 : false,
          description: 'Define to build with systemd support. (-Dsystemd=enabled)')

#
# root-ca-file option
#
cafile = get_option('root-ca-file')
cafile_choices = [
  '/etc/ssl/certs/ca-certificates.crt',
  '/etc/pki/tls/certs/ca-bundle.crt',
  '/etc/ssl/ca-bundle.pem',
  '/etc/pki/tls/cacert.pem',
  '/etc/pki/ca-trust/extracted/pem/tls-ca-bundle.pem',
  '/etc/ssl/cert.pem',
]
if cafile == 'auto'
  foreach f : cafile_choices
    if fs.exists(f)
      cafile = f
      break
    endif
  endforeach
endif
message('root CA certs: @0@'.format(cafile))
cdata.set_quoted('USUAL_TLS_CA_FILE', cafile,
                 description: 'Path to root CA certs.')


cdata.set('USE_EVDNS', use_evdns ? 1 : false,
          description: 'Use libevent for DNS lookups.')

cdata.set('_GNU_SOURCE', 1,
          description: 'Define to get some GNU functions in headers.')

if host_machine.system() == 'windows'
  cdata.set('WIN32_LEAN_AND_MEAN', 1,
            description: 'Define to request cleaner win32 headers.')
  cdata.set('WINVER', '0x0600',
            description: 'Define to max win32 API version (0x0600=Vista).')
endif

cdata.set('CASSERT', get_option('cassert') ? 1 : false,
          description: 'Define to enable assert checking')


anl_dep = cc.find_library('anl', required: false)

os_deps = []

if host_machine.system() == 'windows'
  os_deps += cc.find_library('ws2_32', required: true)
endif

check_funcs = [
  'arc4random_buf',
  'getentropy',
  'getopt',
  'getopt_long',
  'getrandom',
  'inet_ntop',
  'inet_pton',
  'lstat',
  'poll',
  'regcomp',
  'sigaction',
  'strerror_r',
  'strlcpy',
]

foreach func : check_funcs
  found = cc.has_function(func)
  cdata.set('HAVE_' + func.to_upper(), found ? 1 : false,
           description: 'Define to 1 if you have the `@0@\' function.'.format(func))
endforeach

if cc.has_function('getaddrinfo_a', dependencies: [anl_dep])
  cdata.set('HAVE_GETADDRINFO_A', 1)
endif

if threads_dep.found()
  cdata.set('HAVE_PTHREAD', 1)
endif

header_checks = [
  'arpa/inet.h',
  'getopt.h',
  'grp.h',
  'netdb.h',
  'netinet/in.h',
  'netinet/tcp.h',
  'poll.h',
  'pthread.h',
  'pwd.h',
  'regex.h',
  'syslog.h',
  'sys/param.h',
  'sys/resource.h',
  'sys/socket.h',
  'sys/time.h',
  'sys/types.h',
  'sys/ucred.h',
  'sys/un.h',
  'ucred.h',
  'unistd.h',
]

foreach header : header_checks
  varname = 'HAVE_' + header.underscorify().to_upper()

  found = cc.has_header(header)
  cdata.set(varname, found ? 1 : false,
            description: 'Define to 1 if you have the <@0@> header file.'.format(header))
endforeach

if not cc.has_type('uid_t', prefix: '#include <sys/types.h>')
  cdata.set('uid_t', 'int')
endif

if not cc.has_type('gid_t', prefix: '#include <sys/types.h>')
  cdata.set('gid_t', 'int')
endif

if not cc.has_type('pid_t', prefix: '#include <sys/types.h>')
  cdata.set('pid_t', 'int')
endif

cdata.set_quoted('PACKAGE_BUGREPORT', 'https://github.com/pgbouncer/pgbouncer/issues',
                 description: 'Define to the address where bug reports for this package should be sent.')
cdata.set_quoted('PACKAGE_NAME', meson.project_name(),
                 description: 'Define to the full name of this package.')
cdata.set_quoted('PACKAGE_STRING', '@0@ @1@'.format(meson.project_name(), meson.project_version()),
                 description: 'Define to the full name and version of this package.')
cdata.set_quoted('PACKAGE_URL', 'https://www.pgbouncer.org/',
                 description: 'Define to the home page for this package.')
cdata.set_quoted('PACKAGE_VERSION', meson.project_version(),
                 description: 'Define to the version of this package.')


# Descend into subdirectory to set up lib/usual/config.h.
subdir('lib/usual')


pgbouncer_sources = files(
  'src/admin.c',
  'src/client.c',
  'src/dnslookup.c',
  'src/hba.c',
  'src/janitor.c',
  'src/ldapauth.c',
  'src/loader.c',
  'src/messages.c',
  'src/main.c',
  'src/objects.c',
  'src/pam.c',
  'src/pktbuf.c',
  'src/pooler.c',
  'src/proto.c',
  'src/prepare.c',
  'src/sbuf.c',
  'src/scram.c',
  'src/server.c',
  'src/stats.c',
  'src/system.c',
  'src/takeover.c',
  'src/util.c',
  'src/varcache.c',
  'src/common/base64.c',
  'src/common/bool.c',
  'src/common/cryptohash.c',
  'src/common/hmac.c',
  'src/common/pgstrcasecmp.c',
  'src/common/saslprep.c',
  'src/common/scram-common.c',
  'src/common/sha2.c',
  'src/common/string.c',
  'src/common/unicode_norm.c',
  'src/common/wchar.c',
)

if host_machine.system() == 'windows'
  pgbouncer_sources += files('win32/win32support.c')
endif

libusual_sources = files(
  'lib/usual/aatree.c',
  'lib/usual/base.c',
  'lib/usual/cbtree.c',
  'lib/usual/cfparser.c',
  'lib/usual/cxalloc.c',
  'lib/usual/cxextra.c',
  'lib/usual/err.c',
  'lib/usual/fileutil.c',
  'lib/usual/logging.c',
  'lib/usual/mbuf.c',
  'lib/usual/pgutil.c',
  'lib/usual/safeio.c',
  'lib/usual/slab.c',
  'lib/usual/socket.c',
  'lib/usual/string.c',
  'lib/usual/strpool.c',
  'lib/usual/time.c',
  'lib/usual/crypto/csrandom.c',
  'lib/usual/crypto/keccak.c',
  'lib/usual/crypto/keccak_prng.c',
  'lib/usual/crypto/md5.c',
  'lib/usual/tls/tls.c',
  'lib/usual/tls/tls_cert.c',
  'lib/usual/tls/tls_client.c',
  'lib/usual/tls/tls_compat.c',
  'lib/usual/tls/tls_config.c',
  'lib/usual/tls/tls_conninfo.c',
  'lib/usual/tls/tls_ocsp.c',
  'lib/usual/tls/tls_peer.c',
  'lib/usual/tls/tls_server.c',
  'lib/usual/tls/tls_util.c',
  'lib/usual/tls/tls_verify.c',
)

if host_machine.system() == 'windows'
  libusual_sources += files(
    'lib/usual/crypto/chacha.c',
    'lib/usual/crypto/digest.c',
    'lib/usual/crypto/entropy.c',
    'lib/usual/getopt.c',
    'lib/usual/list.c',
    'lib/usual/mempool.c',
    'lib/usual/netdb.c',
    'lib/usual/regex.c',
    'lib/usual/signal.c',
    'lib/usual/socket_ntop.c',
    'lib/usual/socket_pton.c',
  )
endif

pgbouncer = executable('pgbouncer',
           pgbouncer_sources,
           libusual_sources,
           config_h,

           include_directories: ['include', 'lib', 'win32'],
           install: true,
           dependencies: [os_deps, libcares, libevent, libldap, openssl, pam, systemd, threads_dep],
)

pandoc = find_program('pandoc', required: true, native: true)
python = find_program('python3', 'python', required: true, native: true)

subdir('doc')

pkgdocdir = get_option('datadir') / 'doc' / 'pgbouncer'

install_data('README.md',
             'NEWS.md',
             'etc/pgbouncer-minimal.ini',
             'etc/pgbouncer.ini',
             'etc/pgbouncer.service',
             'etc/pgbouncer.socket',
             'etc/userlist.txt',
             install_dir: pkgdocdir)

test('optscan',
     files('etc/optscan.sh'),
     workdir: meson.project_source_root())

test('pytest',
     python,
     args: ['-m', 'pytest', '-n', 'auto', '-r', 's', '-v', '..'],
     env: {
            'BOUNCER_EXE': pgbouncer.full_path(),
            'PYTHONIOENCODING': 'utf8',
            'TEST_USE_LDAP': (libldap.found() ? '1' : ''),
            'TEST_USE_TLS': (openssl.found() ? '1' : ''),
          },
     depends: [pgbouncer],
     timeout: 1000,
    )

subdir('test')


summary({'cares': libcares,
         'evdns': use_evdns,
         'ldap': libldap,
         'openssl': openssl,
         'pam': pam,
         'systemd': systemd,
        }, section: 'Configuration')


#
# dist
# (adapted from PostgreSQL)
#

# Meson has its own distribution building command (meson dist), but we
# are not using that at this point.  The main problem is that, the way
# they have implemented it, it is not deterministic.  Also, we want it
# to be equivalent to the "make" version for the time being.  But the
# target name "dist" in meson is reserved for that reason, so we call
# the custom target "pgdist".

git = find_program('git', required: false, native: true, disabler: true)
gzip = find_program(get_option('GZIP'), native: true, required: false)

distdir = meson.project_name().to_lower() + '-' + meson.project_version()

pg_git_revision = get_option('PG_GIT_REVISION')

extra_dist = ['config.guess', 'config.sub', 'configure', 'install-sh', 'lib/usual/config.h.in']

git_archive_args = []
foreach f : extra_dist
  dirname = fs.parent(f)
  if dirname == '.'
    dirname = ''
  else
    dirname += '/'
  endif
  git_archive_args += ['--prefix', '@0@/@1@'.format(distdir, dirname), '--add-file', f]
endforeach

tar_gz = custom_target('tar.gz',
  build_always_stale: true,
  command: [git, '-C', '@SOURCE_ROOT@',
            '-c', 'core.autocrlf=false',
            'archive',
            '--format', 'tar.gz',
            '-9',
            '--prefix', distdir + '/',
            '-o', join_paths(meson.project_build_root(), '@OUTPUT@'),
            pg_git_revision] + git_archive_args + ['--prefix', distdir + '/'],
  output: distdir + '.tar.gz',
)

alias_target('pgdist', [tar_gz])

# Make the standard "dist" command fail, to prevent accidental use.
# But not if we are in a subproject, in case the parent project wants to
# create a dist using the standard Meson command.
if not meson.is_subproject()
  meson.add_dist_script(python, '-c', 'import sys; sys.exit(1)')
endif
