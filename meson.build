project('bluealsa-monitor', version: '0.0.2', license: 'Unlicense')

monitor_script = 'bluealsa-monitor'

prefix = get_option('prefix')
assert(prefix.startswith('/'), 'Prefix is not absolute: "@0@"'.format(prefix))

bindir = join_paths(prefix, get_option('bindir'))
statedir = join_paths(prefix, get_option('localstatedir'), 'lib', 'bluealsa-monitor')
libdir = join_paths(prefix, get_option('datadir'), 'bluealsa-monitor')
alsa_prefix = run_command('pkg-config', '--variable', 'prefix', 'alsa').stdout().strip()
alsaconfdir = join_paths(alsa_prefix, 'share', 'alsa', 'alsa.conf.d')
systemd = dependency('systemd', required: false)

conf_data = configuration_data()
conf_data.set('version', meson.project_version())
conf_data.set('prefix', prefix)
conf_data.set('bindir', bindir)
conf_data.set('libdir', libdir)
conf_data.set('statedir', statedir)
conf_data.set('monitor_script', monitor_script)

configure_file(
	input : 'bluealsa-monitor.in',
	output : 'bluealsa-monitor',
	configuration : conf_data,
	install: true,
	install_dir: bindir
)

configure_file(
	input : 'monitor-funcs.in',
	output : 'monitor-funcs',
	configuration : conf_data,
	install: true,
	install_dir: libdir
)

configure_file(
	input : 'alsaconf.in',
	output : 'alsaconf',
	configuration : conf_data,
	install: true,
	install_dir: libdir
)

configure_file(
	input : 'pulse.in',
	output : 'pulse',
	configuration : conf_data,
	install: true,
	install_dir: libdir
)

install_data(
	'21-bluealsa_raw.conf',
	install_dir: alsaconfdir,
	install_mode: ['rw-rw-rw-', 'root', 'root']
)

mytouch = find_program('touch')
asoundrc = custom_target(
	'asoundrc',
	output: 'asoundrc',
	command : [mytouch,  '@OUTPUT@'],
	install : true,
	install_dir: statedir,
	install_mode: ['rw-rw-r--', 'root', 'audio']
)

if systemd.found()
	unitdir = join_paths(prefix, 'lib', 'systemd', 'user')
	configure_file(
		input : 'bluealsa-monitor.service.in',
		output : 'bluealsa-monitor.service',
		configuration : conf_data,
		install: true,
		install_dir: unitdir
	)
endif

if get_option('udev').enabled()
	sudoersdir = join_paths(prefix, get_option('sysconfdir'), 'sudoers.d')
	install_data('10-bluealsa-monitor', install_mode: ['r--r-----', 'root', 'root'], install_dir: sudoersdir)
endif
