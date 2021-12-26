project('bluealsa-monitor', version: '0.0.4', license: 'Unlicense')

monitor_script = 'bluealsa-monitor'

prefix = get_option('prefix')
assert(prefix.startswith('/'), 'Prefix is not absolute: "@0@"'.format(prefix))

bindir = join_paths(prefix, get_option('bindir'))
libdir = join_paths(prefix, get_option('datadir'), 'bluealsa-monitor')
alsaconfdir = '/etc/alsa/conf.d'
systemd = dependency('systemd', required: false)

conf_data = configuration_data()
conf_data.set('version', meson.project_version())
conf_data.set('prefix', prefix)
conf_data.set('bindir', bindir)
conf_data.set('libdir', libdir)
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

install_data(
	'bluealsa-monitor-env.conf',
	install_dir: libdir,
	install_mode: ['rw-r--r--', 'root', 'root']
)

configure_file(
	input : 'udev.in',
	output : 'udev',
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
	'21-bluealsa-raw.conf',
	install_dir: alsaconfdir,
	install_mode: ['rw-r--r--', 'root', 'root']
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
