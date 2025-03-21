# VM instance metadata

[VM](vm.md) instance details are available in the metadata service. You can use metadata to provide any keys and values, and then request the metadata from [inside](../operations/vm-info/get-info.md#inside-instance) or [outside](../operations/vm-info/get-info.md#outside-instance) a VM instance. For more information about setting up a metadata service, see [{#T}](../operations/vm-info/get-info.md#metadata-options).

Metadata is also used by apps that launch at VM startup.

## Metadata formats supported inside VM {#metadata-formats}

From inside a VM instance, metadata is available in the following formats:
* [Google Compute Engine](../operations/vm-info/get-info.md#gce-metadata) (not all fields are supported).
* [Amazon EC2](../operations/vm-info/get-info.md#ec2-metadata) (not all fields are supported).

## How to send metadata {#how-to-send-metadata}

You can provide metadata when creating or [updating](../operations/vm-control/vm-update.md#change-metadata) VMs. VM connection data cannot be changed, so it must be provided during VM creation:
* [For a Linux VM](../operations/vm-create/create-linux-vm.md), provide the public SSH key to be able to connect to it.


{% list tabs %}

- CLI

   In the CLI, you can specify metadata in three parameters:
   * `--metadata-from-file` as a file, for example, `--metadata-from-file key=path/to/file`. This is convenient when passing values consisting of multiple strings.
   * `--metadata` is the list of `key-value` pairs separated by commas, for example, `--metadata foo1=bar,foo2=baz`.

      If the value is multiline, use `\n` to split lines: `--metadata user-data="#ps1\nnet user Administrator Passw0rd"`.
   * `--ssh-key` with an SSH key. For Linux VMs only.

      {{ compute-name }} creates a user named `yc-user` and adds the specified SSH key to the list of authorized keys. After the VM is created, you can use this key to connect to it over SSH.

   You can combine these parameters, for example:

   ```bash
   yc compute instance create \
     --name my-instance \
     --metadata-from-file user-data=metadata.yaml \
     --metadata serial-port-enable=1
   ...
   ```

- {{ TF }}

   In the {{ TF }}, you can specify metadata in three ways:
   * As a separate file with user metadata to be processed by the cloud-init agent. Under `metadata`, specify the path to the file with user metadata, such as `cloud-init.yaml`:

      ```hcl
      ...
      metadata = {
        user-data = "${file("cloud-init.yaml")}"
      }
      ...
      ```

      {% cut "Sample contents of the `cloud-init.yaml` file" %}

      ```hcl
      #cloud-config
      users:
        - name: <username>
          groups: sudo
          shell: /bin/bash
          sudo: ['ALL=(ALL) NOPASSWD:ALL']
          ssh-authorized-keys:
            - <SSH_key_contents>
      ```

      {% endcut %}

   * In the `metadata` section, as a line with user metadata. If the value is multiline, use `\n` to split lines. For example:

      ```hcl
      ...
      metadata = {
        user-data = "#cloud-config\nusers:\n  - name: <username>\n    groups: sudo\n    shell: /bin/bash\n    sudo: ['ALL=(ALL) NOPASSWD:ALL']\n    ssh-authorized-keys:\n      - <SSH_key _contents>")}"
      }
      ...
      ```

   * For Linux VMs only. Under `ssh-keys`, specify the username and the SSH key to access Linux VMs. Enter your username and the contents of your SSH key as follows:

      ```hcl
      ...
      metadata = {
        ssh-keys = "<username>:<SSH_key_contents>"
      }
      ...
      ```

      If you are using an out-of-the-box public [image](../concepts/image.md) from {{ marketplace-full-name }}, the specified username does not matter. The key will be assigned to the user specified in the `cloud-init` configuration by default. In different images, these users differ.

      If you don't know the default user, find the string containing `Authorized keys from` in the [serial port output](../operations/vm-info/get-serial-port-output.md). It contains the name of the user assigned the authorized keys.

      If no such string is found, but you see the string `no authorized ssh keys fingerprints found for user`, it means that you incorrectly passed your SSH key. Check the format again or try passing the SSH keys in the `user-data` field.

- API

   In the API, specify the metadata in the `metadata` property as a JSON object, e.g.:

   ```json
   "metadata": {
     "ssh-keys": "ssh-ed25519 AAAAB3Nza... user@example.com",
     "serial-port-enable": "1"
   }
   ```

   To enter a line break in values, use the character combination `\n`.

{% endlist %}

{% note warning %}

The metadata, including the user-defined metadata, is available in unencrypted format. If you place confidential information in the metadata, take measures to protect it (for example, by encrypting it).

{% endnote %}

## Keys processed in public images {#keys-processed-in-public-images}

The list of keys that are processed in {{ yandex-cloud }} public images depends on the operating system.

{% list tabs %}

- Linux

   * `serial-port-enable`: Flag that enables access to the [serial console](../operations/serial-console/index.md). `1` means it is enabled, while `0`, which is the default value, means it is disabled.
   * `user-data`: String with the user metadata to be processed by the [cloud-init](https://cloud-init.io) agent running on a VM instance.

      Cloud-init supports different [formats](https://cloudinit.readthedocs.io/en/latest/topics/format.html) for passing metadata, such as [cloud-config](https://cloudinit.readthedocs.io/en/latest/topics/examples.html). In this format, you can pass SSH keys and indicate which user each key is associated with. To do this, specify them in the `users/ssh-authorized-keys` element.

      {% include [user-data](../../_includes/compute/user-data.md) %}

      To pass this data in the request, replace line breaks with `\n`:

      ```json
      "metadata": {
        "user-data": "#cloud-config\nusers:\n  - name: user\n    groups: sudo\n    shell: /bin/bash\n    sudo: ['ALL=(ALL) NOPASSWD:ALL']\n    ssh-authorized-keys:\n      - ssh-ed25519 AAAAB3Nza......OjbSMRX user@example.com\n      - ssh-ed25519 AAAAB3Nza......Pu00jRN user@desktop"
      }
      ```

   * `ssh-keys`: Key for delivering the SSH key to Linux VMs through {{ TF }}. The key is specified in `<username>:<SSH key content>` forma, e.g., `user:ssh-ed25519 AAC4NzaC1 user@example.com`. If you specify multiple keys, only the first one will be used.

- Windows

   `user-data`: A string with user metadata to be processed by the [Cloudbase-Init](https://cloudbase.it/cloudbase-init/) agent. This agent supports various [data formats](https://cloudbase-init.readthedocs.io/en/latest/userdata.html), such as PowerShell scripts that set the administrator password:

   ```json
   "metadata": {
     "user-data": "#ps1\nnet user Administrator Passw0rd"
   }
   ```

{% endlist %}