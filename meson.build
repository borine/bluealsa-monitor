project('bluealsa-monitor', version: '0.0.1', license: 'Unlicense')

prefix = get_option('prefix')
assert(prefix.startswith('/'), 'Prefix is not absolute: "@0@"'.format(prefix))

bindir = join_paths(prefix, get_option('bindir'))
statedir = join_paths(prefix, get_option('localstatedir'), 'lib', 'bluealsa-monitor')

systemd = dependency('systemd', required: false)

conf_data = configuration_data()
conf_data.set('version', meson.project_version())
conf_data.set('prefix', prefix)
conf_data.set('statedir', statedir)

configure_file(
	input : 'bluealsa-monitor.in',
	output : 'bluealsa-monitor',
	configuration : conf_data,
	install: true,
	install_dir: bindir
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
