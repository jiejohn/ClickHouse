# Configuration Files {#configuration_files}

主要的服务器配置文件是 `config.xml`. 文件存储在 `/etc/clickhouse-server/` 目录.

可以通过`config.xml`文件同级目录 `config.d` 下的 `*.xml` 和 `*.conf` 文件来覆盖各个配置项.

可以为这些配置文件（*.xml和*.conf）的元素指定 `replace` 或 `remove` 属性。

如果两者都未指定，则它以递归方式组合元素的内容，替换重复子元素的值。
If neither is specified, it combines the contents of elements recursively, replacing values of duplicate children.

如果指定 `replace` ，它将用指定的元素替换整个元素。
If `replace` is specified, it replaces the entire element with the specified one.

如果指定 `remove` ，将删除整个元素。
If `remove` is specified, it deletes the element.

这些配置可以定义“substitutions"，如果元素具有 `incl` 属性，则文件中的相应替换将用作值。默认情况下，具有替换的文件的路径是/etc/metrika.xml。这可以在服务器配置中的include_from元素中更改。替换值在此文件的/ yandex / substitution_name元素中指定。如果incl中指定的替换不存在，则将其记录在日志中。要防止ClickHouse记录缺少的替换，请指定optional =“true”属性（例如，macros server_settings / settings.md的设置）
The config can also define "substitutions". If an element has the `incl` attribute, the corresponding substitution from the file will be used as the value. By default, the path to the file with substitutions is `/etc/metrika.xml`. This can be changed in the [include_from](server_settings/settings.md#server_settings-include_from) element in the server config. The substitution values are specified in `/yandex/substitution_name` elements in this file. If a substitution specified in `incl` does not exist, it is recorded in the log. To prevent ClickHouse from logging missing substitutions, specify the `optional="true"` attribute (for example, settings for [macros](#macros) server_settings/settings.md)).

也可以从ZooKeeper执行替换。为此，请指定属性from_zk =“/ path / to / node”。元素值将替换为ZooKeeper中/ path / to / node处节点的内容。您还可以在ZooKeeper节点上放置一个完整的XML子树，它将完全插入到源元素中。
Substitutions can also be performed from ZooKeeper. To do this, specify the attribute `from_zk = "/path/to/node"`. The element value is replaced with the contents of the node at `/path/to/node` in ZooKeeper. You can also put an entire XML subtree on the ZooKeeper node and it will be fully inserted into the source element.

config.xml文件可以指定具有用户设置，配置文件和配额的单独配置。此配置的相对路径在“users_config”元素中设置。默认情况下，它是users.xml。如果省略users_config，则直接在config.xml中指定用户设置，配置文件和配额。
The `config.xml` file can specify a separate config with user settings, profiles, and quotas. The relative path to this config is set in the 'users_config' element. By default, it is `users.xml`. If `users_config` is omitted, the user settings, profiles, and quotas are specified directly in `config.xml`.

此外，users_config可能会覆盖users_config.d目录（例如users.d）和替换中的文件。例如，您可以为每个用户提供单独的配置文件，如下所示：
In addition, `users_config` may have overrides in files from the `users_config.d` directory (for example, `users.d`) and substitutions. For example, you can have separate config file for each user like this:
``` xml
$ cat /etc/clickhouse-server/users.d/alice.xml
<yandex>
    <users>
      <alice>
          <profile>analytics</profile>
            <networks>
                  <ip>::/0</ip>
            </networks>
          <password_sha256_hex>...</password_sha256_hex>
          <quota>analytics</quota>
      </alice>
    </users>
</yandex>
```

对于每个配置文件，服务器在启动时也会生成file-preprocessed.xml文件。 这些文件包含所有已完成的替换和覆盖，它们仅供参考。 如果在配置文件中使用ZooKeeper替换但在服务器启动时ZooKeeper不可用，则服务器从预处理文件加载配置。
For each config file, the server also generates `file-preprocessed.xml` files when starting. These files contain all the completed substitutions and overrides, and they are intended for informational use. If ZooKeeper substitutions were used in the config files but ZooKeeper is not available on the server start, the server loads the configuration from the preprocessed file.

服务器跟踪配置文件中的更改，以及执行替换和覆盖时使用的文件和ZooKeeper节点，并动态重新加载用户和群集的设置。 这意味着您可以在不重新启动服务器的情况下修改群集，用户及其设置。
The server tracks changes in config files, as well as files and ZooKeeper nodes that were used when performing substitutions and overrides, and reloads the settings for users and clusters on the fly. This means that you can modify the cluster, users, and their settings without restarting the server.

[Original article](https://clickhouse.yandex/docs/en/operations/configuration_files/) <!--hide-->
