.. Copyright (C) 2013-2019 Maciej Delmanowski <drybjed@gmail.com>
.. Copyright (C) 2014-2019 <https://debops.org/>
.. SPDX-License-Identifier: GPL-3.0-only

Default variable details
========================

.. include:: ../../../includes/global.rst

Some of ``debops.users`` default variables have more extensive configuration than
simple strings or lists, here you can find documentation and examples for them.

.. only:: html

   .. contents::
      :local:
      :depth: 3


.. _users__ref_accounts:

users__accounts
---------------

The ``users__*_groups`` and ``users__*_accounts`` variables define the UNIX
group and UNIX user accounts which should be managed by Ansible. The
distinctive names can be used to order the UNIX group creation before the
account creation; otherwise both variable sets use the same content.

Examples
~~~~~~~~

.. literalinclude:: examples/manage-groups.yml
   :language: yaml
   :lines: 1,5-

.. literalinclude:: examples/manage-accounts.yml
   :language: yaml
   :lines: 1,5-

.. literalinclude:: examples/manage-resources.yml
   :language: yaml
   :lines: 1,5-

Syntax
~~~~~~

The variables are lists of YAML dictionaries, each dictionary defines an UNIX
group or an UNIX account using specific parameters.

General account parameters
''''''''''''''''''''''''''

``name``
  Required. Name of the UNIX user account to manage. If ``group`` parameter is
  not specified, this value is also used to create a private UNIX group for
  a given user. Configuration entries with the same ``name`` parameter are
  merged in order of appearance, this can be used to modify existing
  configuration entries conditionally.

``user``
  Optional, boolean. If not specified or ``True``, a configuration entry will
  manage both an UNIX account and its primary UNIX group. If ``False``, only
  the UNIX group is managed; this can be used to define shared system groups.
  You can also use the :ref:`debops.system_groups` Ansible role to define UNIX
  groups with additional functionality like :command:`sudo` configuration, etc.

``local``
  Optional, boolean. If not specified or ``True``, the role will use the
  ``libuser`` library to manage the UNIX groups and accounts in the local
  account and group database. If ``False``, the role will use standard UNIX
  tools to manage accounts, which might have unintended effects. On normal
  operation you shouldn't need to define this parameter, it's enabled or
  disabled by the role as needed.

  See :ref:`users__ref_libuser` for more details.

``system``
  Optional, boolean. If ``True``, a given user account and primary group will
  be a "system" account and group, with it's UID and GID < 1000. If the value
  is not specified or ``False``, the user account and group will be a "normal"
  account and group with UID and GID >= 1000.

``chroot``
  Optional, boolean. If defined and ``True``, a given user account is
  configured to support SFTPonly operation, and certain defaults are changed if
  not overridden by other parameters.

  The owner of the home directory will be the ``root`` account instead of the
  user, to allow chrooting to that directory, the home directory group will be
  the primary group of a given user. The default permissions are set to
  ``0751``.

  The default shell is set based on the :envvar:`users__chroot_shell` variable,
  by default it will be :file:`/usr/sbin/nologin`. Any dotfiles configured
  globally or for that UNIX account are not installed due to permission issues
  in the home directory.

  The account will be added to UNIX groups specified in the
  :envvar:`users__chroot_groups` variable, by default ``sftponly``. See the
  :ref:`debops.sshd` role for details about configuring the SFTPonly access in
  OpenSSH server.

``uid``
  Optional. Specify the UID of the UNIX user account.

``gid``
  Optional. Specify the GID of the primary group for a given user account.

``group``
  Optional. Name of the UNIX group which will be set as the primary group of
  a given account. If ``group`` is not specified, ``name`` will be used
  automatically to create the corresponding UNIX group.

``private_group``
  Optional, boolean. If specified and ``False``, the role will not try to
  directly manage the specified UNIX ``group`` used with a given UNIX account.
  This is useful if you want to set a primary UNIX group that's used in other
  places and which you might not want to remove with the UNIX account.

``groups``
  Optional. List of UNIX groups to which a given UNIX account should belong.
  Only existing groups will be added to the account.

``append``
  Optional, boolean. If ``True`` (default), the specified groups will be added
  to the list of existing groups the account belongs to. If ``False``, all
  other groups than those present on the group list will be removed stripped.

``comment``
  Optional. A comment, or GECOS field configured for a specified UNIX account.

``shell``
  Optional. Specify the default shell to run when a given UNIX account logs in.
  If not specified, the default system shell (usually :file:`/bin/sh` will be
  used instead).

``password``
  Optional. Specify the encrypted hash of the user's password which will be set
  for a given UNIX account. You can use the ``lookup("password")`` lookup to
  generate the hash. See examples for more details.

``update_password``
  Optional. If set to ``on_create``, the password will be set only one on
  initial user creation. If set to ``always``, the password will be updated on
  each Ansible run if it's different.

  The module default is to always update the password, the ``debops.users``
  default is to only update the password on initial user creation.

``no_log``
  Optional, boolean. If defined and ``True``, a given entry will not be logged
  during the Ansible run. If not specified, if the ``password`` parameter is
  specified, the role will automatically disable logging as well.

``non_unique``
  Optional, boolean. If ``True``, allows setting the UID to a non-unique value.

``linger``
  Optional, boolean. If ``True``, the UNIX account will be allowed to linger
  when not logged in and manage private services via it's own
  :command:`systemd` user instance. If ``False``, the linger option will be
  disabled.

Parameters related to account state
'''''''''''''''''''''''''''''''''''

``state``
  Optional. If ``present``, the UNIX user account and primary group will be
  created. If ``absent``, the specified account and group will be removed.

``force``
  Optional, boolean. If used with ``state`` parameter being ``absent``, Ansible
  will execute the ``userdel --force`` command.

``remove``
  Optional, boolean. If used with ``state`` parameter being ``absent``, Ansible
  will execute the ``userdel --remove`` command.

``expires``
  Optional. Specify the time in the UNIX epoch format, at which a given UNIX
  user account will be disabled.

Parameters related to home directories
''''''''''''''''''''''''''''''''''''''

``home``
  Optional. Path to the home directory of a given user account. If not
  specified, the role will check the home directory path of an existing account
  defined on the host.

  The :ref:`debops.users` role does not create parent directories of home
  directories. If you try to create a home directory in a non-existent
  subdirectory, Ansible will fail. This might be problematic due to the role
  order in the playbook.

  You can use the :ref:`debops.fhs` role to ensure that the base directories
  exist before creating home directories in them. For example, a common
  practice is creation of the web application home directories inside of the
  :file:`/srv/www/` subdirectory which doesn't exist by default. The
  :ref:`debops.fhs` role will create it automatically - it is included in the
  DebOps :file:`common.yml` playbook, and you can include it in custom
  playbooks if needed. See the :ref:`fhs__ref_directories` documentation for
  more details.

``home_owner``
  Optional. Specify the owner of the home directory of a given UNIX account.

``home_group``
  Optional. Specify the group of the home directory of a given UNIX account.

``home_mode``
  Optional. Specify the mode of the home directory of a given UNIX account. If
  not specified, the value of the :envvar:`users__default_home_mode` will be
  used instead.

``create_home``
  Optional, boolean. If ``True``, the role will create the home directory for
  a given user account if it doesn't exist already. If not specified, home
  directory is created by default by the `Ansible ansible.builtin.user module`_.

``move_home``
  Optional, boolean. If ``True`` and the managed user account already exists,
  Ansible will try to move it's home directory to the location specified in the
  ``home`` parameter if it isn't there already.

``skeleton``
  Optional. Specify path to the directory, contents of which will be copied to
  the newly created home directory.

``home_acl``
  Optional. Configure filesystem ACL entries of the home directory of a given
  UNIX user account. This parameter is a list of YAML dictionaries, each
  element uses a specific set of parameters derived from the ``acl`` Ansible
  module, see its documentation for details, as well as the :man:`acl(5)`,
  :man:`setfacl(1)` and :man:`getfacl` manual pages. Some useful parameters:

  ``default``
    Optional, boolean. If ``True``, set a given ACL entry as the default for
    new files and directories inside a given directory. Only works with
    directories.

  ``entity``
    Name of the UNIX user account or group that a given ACL entry applies to.

  ``etype``
    Specify the ACL entry type to configure. Valid choices: ``user``,
    ``group``, ``mask``, ``other``.

  ``permissions``
    Specify the permission to apply for a given ACL entry. This parameter
    cannot be specified when the state of an ACL entry is set to ``absent``.

  ``recursive``
    Apply a given ACL entry recursively to all entities in a given path.

  ``state``
    Optional. If not specified or ``present``, the ACL entry will be created.
    If ``absent``, the ACL entry will be removed. The ``query`` state doesn't
    make sense in this context and shouldn't be used.

Parameters related to the account's private SSH key
'''''''''''''''''''''''''''''''''''''''''''''''''''

``generate_ssh_key``
  Optional, boolean. If ``True``, Ansible will generate a private SSH key for
  the specified account.

``ssh_key_bits``
  Optional. Number of bits to use for the user's private SSH key. If not
  specified, role will use the `Ansible ansible.builtin.user module`_ default
  value.

``ssh_key_comment``
  Optional. Add a custom comment to the generated SSH key.

``ssh_key_file``
  Optional. Path where the private SSH key will be stored.

``ssh_key_passphrase``
  Optional. Set a passphrase which will be required to decrypt the private SSH
  key.

``ssh_key_type``
  Optional. Specify the SSH key type to generate. If not specified, RSA keys
  will be generated automatically.

Parameters related to public SSH keys
'''''''''''''''''''''''''''''''''''''

``sshkeys``
  Optional. String or a YAML list of public SSH keys to configure for a given
  user account. The keys will be stored in the ``~/.ssh/authorized_keys``
  file.

``sshkeys_exclusive``
  Optional, boolean. If ``True``, the role will remove all keys from the user's
  ``~/.ssh/authorized_keys`` file that are not specified in the ``sshkeys``
  parameter.

``sshkeys_follow``
  Optional, boolean. If ``True``, the role will follow symlinks to the user's
  ``~/.ssh/authorized_keys`` file instead of replacing them.

``sshkeys_state``
  Optional. If not specified or ``present``, the SSH keys will be set on the
  user's account. If ``absent``, the ``~/.ssh/authorized_keys`` file will be
  removed entirely.

Parameters related to mail forwarding
'''''''''''''''''''''''''''''''''''''

``forward``
  Optional. String or YAML list of e-mail addresses which will be used to
  forward mail directed to a given UNIX account. They will be stored in the
  ``~/.forward`` file. This is only valid for MTAs that support this mechanism,
  for example Postfix MTA when local mail is enabled.

``forward_state``
  Optional. If not specified or ``present``, the e-mail addresses specified in
  the ``forward`` parameter will be added to the ``~/.forward`` configuration
  file. If ``absent``, the entries will be removed from the configuration file.

Parameters related to user configuration files
''''''''''''''''''''''''''''''''''''''''''''''

``dotfiles_enabled`` / ``dotfiles``
  Optional, boolean. Enable or disable management of the user configuration
  files.

``dotfiles_repo``
  Optional. An URL or an absolute path on the host to the :command:`git`
  repository with the user configuration files to deploy. If not specified, the
  default dotfiles repository, defined in the :envvar:`users__dotfiles_repo`
  variable, will be used instead. The repository will be deployed or updated
  using the :command:`yadm` script, installed by the :ref:`debops.yadm` Ansible
  role.

Parameters related to directory and file resources
''''''''''''''''''''''''''''''''''''''''''''''''''

``resources``
  This parameter can be used to manage directories, files and symlinks for
  specific UNIX accounts using Ansible inventory. This functionality is meant to
  be used to manage small amounts of data, like custom configuration files,
  private SSH keys and so on. For more advanced management, you should consider
  using :ref:`debops.resources` Ansible role, or even writing a custom Ansible
  role from scratch.

  Tasks that manage the resources are executed as the ``root`` account, but the
  owner and group of the files is automatically set to those used by a given UNIX
  account. Directory and file paths will be prepended with a path to the
  ``$HOME`` directory of a given user, and should be defined as relative, without
  ``/`` at the beginning.

  The ``resources`` parameter should contain a list of entries, each entry should
  be defined as either a path string which denotes a directory relative to the
  user's ``$HOME`` directory, or a YAML dictionary that describes a given
  resource using specific parameters:

  ``dest`` or ``path``
    Required. Path to the resource managed by this entry, relative to the user's
    ``$HOME`` directory. All subdirectories specified in the path will be created
    automatically.

  ``content``
    If the resource type is a ``file``, this parameter can be used to specify the
    contents of the file that is managed by this entry, usually in the form of
    a YAML text block. It shouldn't be specified together with the ``src``
    parameter.

  ``src``
    If the resource type is a ``link``, this parameter specifies the target of
    the symlink. In case of symlinks to resources owned by other UNIX accounts
    than the user, you need to specify the ``owner`` and ``group`` parameters to
    that of the symlinked file (for example ``root`` for files or directories
    owned by the ``root`` account), otherwise the role will change them to the
    owner/group of a given user.

    If the resource type is a ``file``, this parameter can be used to specify the
    source file on the Ansible Controller to copy to the remote host. It
    shouldn't be specified together with the ``content`` parameter.

  ``state``
    Optional. This variable defines the resource state and it's type:

    - ``absent``: the resource will be removed
    - ``directory``: the resource is a directory
    - ``file``: the resource is a file
    - ``link``: the resource is a symlink
    - ``touch``: the resource will create an empty file, or "touch" an existing
      file on each Ansible run

    If this parameter is not specified, the resource will be treated as
    a directory.

  ``force``
    Optional, boolean. If ``True``, the files will be always overwritten, if
    ``False``, files will be copied only if they don't exist. This parameter can
    also be used to force creation of symlinks.

  ``owner``
    Optional. Specify the UNIX account which should be the owner of a given
    file/directory. For symlinks, this defines the owner of the link source and
    might be needed if the owner is different than the current user.

  ``group``
    Optional. Specify the UNIX group which should be the primary group of
    a given file/directory. For symlinks, this defines the group of the link
    source and might be needed if the group is different than the primary group
    of the current user.

  ``mode``
    Optional. Set specific permissions for a given file/directory/symlink.

  ``recurse``
    Optional, boolean. Recursively set specified permission for all directories
    in the directory tree that lead to a given directory/file, depending on user
    privileges.

  ``parent_owner``
    Optional. Specify the UNIX account that should be the owner of a parent
    directory of a given resource.

  ``parent_group``
    Optional. Specify the UNIX group that should be the main group of a parent
    directory of a given resource.

  ``parent_mode``
    Optional. Specify the permissions of the parent directory of a given
    file resource.

  ``parent_recurse``
    Optional, boolean. If ``True``, parent permissions will be applied
    recursively to all parent directories.
