---
title: 1 - 自定义Charts商店
weight: 1
aliases:
  - /rancher/v2.x/en/tasks/global-configuration/catalog/customizing-charts/
---

## Chart目录结构

下表说明了Chart 的目录结构，可以在 Chart目录中找到: `charts/%application%/%app version%/`. 在为自定义应用商店创建自定义Chart 时，此信息非常有用. 用**Rancher Specific**表示的文件特定于Rancher charts，但对于charts自定义是可选的。

| 目录         | 文件               | 描述                                                         |
| ------------ | ------------------ | ------------------------------------------------------------ |
|              | `app-readme.md`    | **Rancher Specific：** Rancher UI中Chart标题中显示的文本。   |
| `charts/`    |                    | 包含依赖关系的目录                                           |
|              | `Chart.yml`        | 必需的Helm信息文件。                                         |
|              | `questions.yml`    | **Rancher Specific：**包含在Rancher UI中显示的表单问题的文件。问题显示在**配置选项中**。 |
|              | `README.md`        | 可选：Rancher UI中显示的Helm自述文件。此文本显示在**详细说明中**。 |
|              | `requirements.yml` | 可选的YAML文件，列出Chart的依赖关系。                        |
| `templates/` |                    | 模板目录，当与`values.yml`结合使用时，生成Kubernetes YAML。  |
|              | `values.yml`       | Chart的默认配置值。                                          |

## Rancher Chart 附加文件

在创建自己的自定义catalog之前，您应该了解基本的Rancher Chart与本机Helm chart的不同之处。Rancher chart与目录结构中的Helm chart略有不同，Rancher chart包括Helm chart没有的两个文件。

- `app-readme.md`

    在chart的UI标题中提供描述性文本的文件。下图显示了Rancher chart（包括`app-readme.md`）和本机Helm chart（不包括）之间的差异。

    Rancher chart`app-readme.md`（左）与Helm chart没有（右）

    <small>Rancher Chart with <code>app-readme.md</code> (left) vs. Helm Chart without (right)</small>

    ![app-readme.md]({{< baseurl >}}/img/rancher/app-readme.png)

- `questions.yml`

    包含表单问题的文件。这些问题简化了chart的部署。没有它，您必须使用键值对配置部署，这更加困难。下图显示了Rancher chart（包括`questions.yml`）和本机Helm chart（不包括）之间的差异。

    Rancher Chart`questions.yml`（左）与Helm Chart没有（右）


	<small>Rancher Chart with <code>questions.yml</code> (left) vs. Helm Chart without (right)</small>
	
	![questions.yml]({{< baseurl >}}/img/rancher/questions.png)

### 问题变量参考

此引用包含可以使用的变量`questions.yml`.

| **变量** | 类型 | 要求 | 描述 |
| ------------- | ------------- | --- |------------- |
| 	variable          | string  | true    | 定义`values.yml`文件中指定的变量名称，`foo.bar`用于嵌套对象。 |
| 	label             | string  | true      | 定义UI标签。 |
| 	description       | string  | false      | 指定变量的描述。 |
| 	type              | string  | false      | 如果没有指定，默认为`string`(当前支持的类型是string、boolean、int、enum、password、storageclass和主机名)。 |
| 	required          | bool    | false      | 定义是否需要该变量(true |
| 	default           | string  | false      | 指定默认值。 |
| 	group             | string  | false      | 按输入值分组问题。 |
| 	min_length        | int     | false      | 最小字符长度。 |
| 	max_length        | int     | false      | 最大字符长度。 |
| 	min               | int     | false      | 最小整数长度。 |
| 	max               | int     | false      | 最大整数长度。 |
| 	options           | []string | false     | 指定变量类型为`enum`时的选项，例如:options: <br/>"ClusterIP" <br> - "NodePort" <br> - "LoadBalancer" |
| 	valid_chars       | string   | false     | 输入字符验证的正则表达式。 |
| 	invalid_chars     | string   | false     | 无效输入字符验证的正则表达式。 |
| 	subquestions      | []subquestion | false| 添加子问题数组。 |
| 	show_if           | string      | false  | 如果条件变量为true，则显示当前变量。例如: `show_if: "serviceType=Nodeport"` |
| 	show\_subquestion_if |  string  | false     | 如果为真或等于其中一个选项，则显示子问题。例如`show_subquestion_if: "true"` |

>**Note:** `subquestions[]`不能包含`subquestions`或`show_subquestions_if`键，但支持上表中的所有其他键。

## 自定义Chart示例

 您可以使用Helm Charts或Rancher Charts填充您的自定义catalog，但我们建议使用Rancher Charts增强用户体验。

>**Note:** 有关开发Charts的完整教程，请参考Helm [developer reference](https://docs.helm.sh/developing_charts/).

1. 在您用作自定义应用商店的GitHub存储库中, 根据 [chart目录结构](#chart目录结构)创建chart目录结构

    Rancher需要这种目录结构，但`app-readme.md`和`questions.yml`是可选的。

2. **推荐：**创建一个`app-readme.md`文件。

    使用此文件在Rancher UI中为Charts标题创建自定义文本。您可以使用此文本告诉用户该Charts是针对您的环境自定义的，或者提供有关如何使用它的特殊说明。 

    **示例**:

    ```bash
    $ cat ./app-readme.md
    
    # Wordpress ROCKS!
    ```

3. **推荐：**创建一个`questions.yml`文件。

    此文件为用户创建表单，以便在部署自定义Charts时指定部署参数。如果没有此文件，用户**必须**使用键值对手动指定参数。

    下面的示例创建一个表单，提示用户输入持久卷大小和存储类。 \

    有关可在创建`questions.yml`文件时使用的变量列表，请参阅[问题变量参考](#问题变量参考)。

        
        categories:
        - Blog
        - CMS
        questions:
        - variable: persistence.enabled
        default: "false"
        description: "Enable persistent volume for WordPress"
        type: boolean
        required: true
        label: WordPress Persistent Volume Enabled
        show_subquestion_if: true
        group: "WordPress Settings"
        subquestions:
        - variable: persistence.size
            default: "10Gi"
            description: "WordPress Persistent Volume Size"
            type: string
            label: WordPress Volume Size
        - variable: persistence.storageClass
            default: ""
            description: "If undefined or null, uses the default StorageClass. Default to null"
            type: storageclass
            label: Default StorageClass for WordPress
        
