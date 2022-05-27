buildiso
========

This role makes remastering Ubuntu images easier. There is support for copying your own files to the installation media, and to sync Personal Package Archive(s) (PPA) from Launchpad to the image.

Requirements
------------

The role requires the `isohybrid`, `xorriso` and `mksquashfs` binaries (the latter if you use the PPA sync functionality). If you are on a version of Ubuntu later than 21.10, you will likely want to install `cd-boot-images-<architecture>`, unless you've overridden `buildiso_efi` and `buildiso_grub` with your own locations.

Role Variables
--------------

Available variables are listed below, along with default values (see `defaults/main.yml`, `tasks/main.yml` and `vars/main.yml`):

Which Ubuntu image to fetch and customise, and where to place these images. The filename can be added to `buildiso_input` for backwards compatibility, but will otherwise be found from `buildiso_pattern` in conjunction with the SHA256SUMS file found in the `buildiso_input` directory.

    buildiso_arch: amd64
    buildiso_input: "http://releases.ubuntu.com/{{ ansible_distribution_release }}/"
    buildiso_pattern: ".*-{{ buildiso_type }}-{{ buildiso_arch }}.iso$"
    buildiso_output: ./custom.iso
    buildiso_type: desktop

You can create or alter `/preseed/<item.name>.seed` by following these examples. Remember that `content` and `src` is mutually exclusive. You likely want to create a `buildiso_bootmenu` to use these.

    buildiso_preseed:
      - name: test1
        content:
          # Your preseed file
      - name: test2
        src: path/to/file

The boot menu of the image can be set to alternative options. The first entry will be the default. `preseed`, `kernel` and `initrd` have default values defined.

`buildiso_bootmenu_hd` provides the option to boot from the first hard disk, and `buildiso_bootmenu_memtest` lets the user start a memory test.
`buildiso_bootmenu_bios` will active the option to change your BIOS settings (fwsetup in UEFI). All of those options defaults to `True`.

There is also `builtiso_bootmenu_timeout`, which defaults to 5 (seconds).

    buildiso_bootmenu:
      - name: '^Try Ubuntu without installing'
        label: live
        kernel: /casper/vmlinuz
        initrd: /casper/initrd
        preseed: file=/cdrom/preseed/ubuntu.seed

      - name: '^Install Ubuntu'
        label: live-install
        cmdline: maybe-ubiquity

      - name: 'OEM install (for manufacturers)'
        cmdline: 'only-ubiquity'
        pkgconf: 'oem-config/enable=true'

      - name: 'Check disc for defects'
        label: check
        cmdline: integrity-check

List of files to copy on to the image, if wanted. You only need to specify `src` if you want the basename of that filename as `dest`. If you put a directory as `src`, the contents will end up in `/extras/`.

    buildiso_files:
      - src: files/
      - src: /etc/resolv.conf
        dest: resolvconf
        mode: preserve

There are a few variables for generating the ISO file. `buildiso_name` variable sets the volume ID, the rest should have sensible defaults.

    buildiso_boot: "{{ 'boot/grub/i386-pc/eltorito.img' if ansible_distribution_version is version('21.10', 'ge') else 'isolinux/isolinux.bin' }}"
    buildiso_cat: "{{ 'boot.catalog' if ansible_distribution_version is version('21.10', 'ge') else 'isolinux/boot.cat' }}"
    buildiso_efi: "{{ '/usr/share/cd-boot-images-amd64/images/boot/grub/efi.img' if ansible_distribution_version is version('21.10', 'ge')
                       else buildiso_writable + '/boot/grub/efi.img' }}"
    buildiso_grub: "{{ '/usr/share/cd-boot-images-amd64/images/boot/grub/i386-pc/boot_hybrid.img' if ansible_distribution_version is version('21.10', 'ge')
                        else False }}"
    buildiso_name: Ubuntu

If something went wrong, but the role will not pick up enough changes to actually build the images, you can now define `buildiso_forcebuild` to make it always run the build tasks.

When you require extra packages, this role provides support for syncing a PPA(s) (Personal Package Archive) if the `buildiso_ppa` variable is set.
It uses the standard `ppa:team/ppa` structure, either a string or a list of strings. See the example playbook to see how you would synchronise the Ansible Stable PPA.

Role Tags
---------

If you specify the `cleanup` tag when running `ansible-playbook`, we will unmount and remove all the temporary directories.

Example Playbook
----------------

    - hosts: localhost
      become: true
      roles:
        - { role: vcc_caeit.build_iso, buildiso_ppa: ppa:ansible/ansible }

License
-------

GPLv2

Author Information
------------------

This role was created in 2019 by Nafallo Bjälevik, whilst doing consultancy work for [Volvo Cars](http://www.volvocars.com/).
