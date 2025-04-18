# SPDX-License-Identifier: LGPL-2.1-or-later

integration_test_wrapper = find_program('integration-test-wrapper.py')
integration_tests = []
integration_test_template = {
        'mkosi-args' : [],
        'timeout' : 1800,
        'storage' : 'volatile',
        'priority' : 0,
        'firmware' : 'linux-noinitrd',
        'enabled' : true,
        'configuration' : {
                'memory-accounting' : 'no',
                'command' : '/usr/lib/systemd/tests/testdata/units/%N.sh',
                'wants' : 'multi-user.target user@4711.service',
                'after' : 'user@4711.service',
                'env' : {},
        },
        'cmdline' : [],
        'credentials' : [],
        'qemu-args' : [],
        'exit-code' : 123,
        'vm' : false,
        'coredump-exclude-regex' : '',
        'sanitizer-exclude-regex' : '',
        'rtc' : false,
}

foreach dirname : [
        'TEST-01-BASIC',
        'TEST-02-UNITTESTS',
        'TEST-03-JOBS',
        'TEST-04-JOURNAL',
        'TEST-05-RLIMITS',
        'TEST-06-SELINUX',
        'TEST-07-PID1',
        'TEST-08-INITRD',
        'TEST-09-REBOOT',
        'TEST-13-NSPAWN',
        'TEST-15-DROPIN',
        'TEST-16-EXTEND-TIMEOUT',
        'TEST-17-UDEV',
        'TEST-18-FAILUREACTION',
        'TEST-19-CGROUP',
        'TEST-21-DFUZZER',
        'TEST-22-TMPFILES',
        'TEST-23-UNIT-FILE',
        'TEST-24-CRYPTSETUP',
        'TEST-25-IMPORT',
        'TEST-26-SYSTEMCTL',
        'TEST-29-PORTABLE',
        'TEST-30-ONCLOCKCHANGE',
        'TEST-31-DEVICE-ENUMERATION',
        'TEST-32-OOMPOLICY',
        'TEST-34-DYNAMICUSERMIGRATE',
        'TEST-35-LOGIN',
        'TEST-36-NUMAPOLICY',
        'TEST-38-FREEZER',
        'TEST-43-PRIVATEUSER-UNPRIV',
        'TEST-44-LOG-NAMESPACE',
        'TEST-45-TIMEDATE',
        'TEST-46-HOMED',
        'TEST-50-DISSECT',
        'TEST-52-HONORFIRSTSHUTDOWN',
        'TEST-53-ISSUE-16347',
        'TEST-54-CREDS',
        'TEST-55-OOMD',
        'TEST-58-REPART',
        'TEST-59-RELOADING-RESTART',
        'TEST-60-MOUNT-RATELIMIT',
        'TEST-62-RESTRICT-IFACES',
        'TEST-63-PATH',
        'TEST-64-UDEV-STORAGE',
        'TEST-65-ANALYZE',
        'TEST-66-DEVICE-ISOLATION',
        'TEST-67-INTEGRITY',
        'TEST-68-PROPAGATE-EXIT-STATUS',
        'TEST-69-SHUTDOWN',
        'TEST-70-TPM2',
        'TEST-71-HOSTNAME',
        'TEST-72-SYSUPDATE',
        'TEST-73-LOCALE',
        'TEST-74-AUX-UTILS',
        'TEST-75-RESOLVED',
        'TEST-76-SYSCTL',
        'TEST-78-SIGQUEUE',
        'TEST-79-MEMPRESS',
        'TEST-80-NOTIFYACCESS',
        'TEST-81-GENERATORS',
        'TEST-82-SOFTREBOOT',
        'TEST-83-BTRFS',
        'TEST-84-STORAGETM',
        'TEST-85-NETWORK',
        'TEST-86-MULTI-PROFILE-UKI',
        'TEST-87-AUX-UTILS-VM',
]
        subdir(dirname)
endforeach

foreach integration_test : integration_tests
        integration_test_args = [
                # We don't use meson.project_source_root() because that doesn't work for running the tests
                # standalone (see standalone/meson.build).
                '--meson-source-dir', meson.current_source_dir() / '../..',
                '--meson-build-dir', meson.project_build_root(),
                '--name', integration_test['name'],
                '--storage', integration_test['storage'],
                '--firmware', integration_test['firmware'],
                '--exit-code', integration_test['exit-code'].to_string(),
                '--coredump-exclude-regex', integration_test['coredump-exclude-regex'],
                '--sanitizer-exclude-regex', integration_test['sanitizer-exclude-regex'],
        ]

        if 'unit' in integration_test
                integration_test_args += ['--unit', fs.name(integration_test['unit'])]
        else
                integration_test_args += ['--unit', '@0@.service'.format(integration_test['name'])]
        endif

        if integration_test['vm']
                integration_test_args += ['--vm']
        endif

        if integration_test['rtc']
                integration_test_args += ['--rtc']
        endif

        if not integration_test['enabled']
                integration_test_args += ['--skip']
        endif

        if not mkosi.found()
                continue
        endif

        integration_test_args += ['--mkosi', mkosi.full_path(), '--']

        if integration_test['cmdline'].length() > 0
                integration_test_args += [
                        '--kernel-command-line-extra=@0@'.format(' '.join(integration_test['cmdline']))
                ]
        endif

        foreach credential : integration_test['credentials']
                integration_test_args += ['--credential', credential]
        endforeach

        if integration_test['qemu-args'].length() > 0
                integration_test_args += ['--qemu-args=@0@'.format(' '.join(integration_test['qemu-args']))]
        endif

        integration_test_args += integration_test['mkosi-args']

        integration_test_env = {}

        if want_integration_tests
                integration_test_env += {'SYSTEMD_INTEGRATION_TESTS': '1'}
        endif

        # We don't explicitly depend on the "mkosi" target because that means the image is rebuilt on every
        # "ninja -C build". Instead, the mkosi target has to be rebuilt manually before running the
        # integration tests with mkosi.
        test(
                integration_test['name'],
                integration_test_wrapper,
                env : integration_test_env,
                args : integration_test_args,
                timeout : integration_test['timeout'],
                priority : integration_test['priority'],
                suite : 'integration-tests',
        )
endforeach
