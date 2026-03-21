# Clash Verge 扩展脚本以及使用教程

受够了机场自带配置里几十个花里胡哨、乱七八糟的代理组？经常不知道该选哪个节点？

用这段扩展脚本，把你的 Clash Verge 变成清爽、高效的精简形态！

配置完成后，你的主界面将像下面这样：
*   🚀 **区域切换**（你想连接的国家/地区）
*   例：**🇨🇳 香港...等**（你建立的目标国家/地区代理组）
*   **🇨🇳 国内直连**（国内网站直连）
*   **🚫 广告拦截**（自动拦截广告）
*   **🌍 全球总库**（所有节点集合）

---

## 部署指南：

### 第一步：编辑并注入“扩展脚本”
1. 打开 **Clash Verge**，点击左侧导航栏的“订阅”。
2. 找到你想要修改的机场订阅，右键点击它，选择“编辑文件”，下滑找到诸如 `- { name: '🌏 国内网站',` 以及 `- { name: '广告拦截',` 的代码行。
3. 将“🌏 国内网站”以及“广告拦截”名称完整复制进下方代码中的“直连代理组名单”以及“广告拦截组名单”中。完成后点击“取消”退出编辑（防止改动原代码）。
4. 再次右键点击刚才的机场订阅，选择 **“扩展脚本”**。
5. 清空里面的原有内容，将下方这段完整的代码全部复制并粘贴进去，然后点击右下角的“保存”：

### 配置文件
<details>
<summary>点击展开查看并复制代码</summary>


```javascript
function main(config) {

  // 直连代理组名单（请在此添加原配置中的直连代理组名称）
  const directList = ["🌏 国内网站", "CN"]; 
  
  // 广告拦截组名单（若原配置有去广告组，请在此添加原配置中的广告拦截组名称）
  const adblockList = ["广告拦截", "AdBlock"];
  
  // 标识符 (默认为特殊空白符“U+FFA0”)
  // 软件内添加代理组后，请精准复制括号内的隐藏字符友军识别标识符（双击即可选中）：`ﾠ`，粘贴到代理组名称的尾部，例如：“🇨🇳 香港ﾠ”
  const watermark = "\uFFA0";

  const masterBucketName = "🚀 区域切换";
  const directBucketName = "🇨🇳 国内直连";
  const adblockBucketName = "🚫 广告拦截";
  const globalBucketName = "🌍 全球总库" + watermark;

  const proxies = config.proxies || [];
  const proxyNames = proxies.map(p => p.name);

  const existingGroups = config["proxy-groups"] || [];
  const keptGroups = existingGroups.filter(g => g.name.includes(watermark));
  const keptGroupNames = keptGroups.map(g => g.name);

  const globalGroup = {
    name: globalBucketName,
    type: "select",
    proxies: proxyNames.length > 0 ? proxyNames : ["DIRECT"]
  };

  const masterGroup = {
    name: masterBucketName,
    type: "select",
    proxies: [...keptGroupNames, globalBucketName]
  };

  const directGroup = {
    name: directBucketName,
    type: "select",
    proxies: ["DIRECT", masterBucketName]
  };

  const adblockGroup = {
    name: adblockBucketName,
    type: "select",
    proxies: ["REJECT", "DROP", masterBucketName, "DIRECT"]
  };

  config["proxy-groups"] = [
    masterGroup,
    ...keptGroups, 
    directGroup,
    adblockGroup,
    globalGroup
  ];

  if (config.rules && Array.isArray(config.rules)) {
    config.rules = config.rules.map(rule => {
      let parts = rule.split(',');
      let targetIndex = -1;

      if (parts.length >= 3 && !rule.trim().startsWith("MATCH") && !rule.trim().startsWith("FINAL")) {
        targetIndex = 2;
      } 
      else if ((rule.trim().startsWith("MATCH") || rule.trim().startsWith("FINAL")) && parts.length >= 2) {
        targetIndex = 1;
      }

      if (targetIndex !== -1) {
        let originalTarget = parts[targetIndex].trim();

        if (originalTarget === "DIRECT" || originalTarget === "REJECT" || originalTarget === "DROP") {
        }
        else if (directList.includes(originalTarget)) {
          parts[targetIndex] = directBucketName;
        }
        else if (adblockList.includes(originalTarget)) {
          parts[targetIndex] = adblockBucketName;
        }
        else {
          parts[targetIndex] = masterBucketName;
        }
      }
      
      return parts.join(',');
    });
  }

  return config;
}
```
</details>

或者[点击此处下载配置文件](./Clash-Verge-Script.js)。

### 第二步：建立你的目标地区组
这段代码会消除机场自带的杂乱分组，同时会保留你自己建立的分组。为了防止建立的分组被消除，我们需要在你建立的分组后加入一段友军识别标识符。

1. 在 Clash Verge 中新建代理组（比如你想建一个只放美国节点的组）。
2. 点击左侧导航栏的“订阅”。
3. 找到你想要修改的机场订阅，右键点击它，选择“编辑代理组”。
4. 在命名时（代理组组名），请**务必**在名字的最末尾，粘贴这个友军识别标识符：“`ﾠ`”。*（请选中引号里的空白处进行复制，它是一个特殊填补符）*

**命名示范：**
*   **正确**：“`🇨🇳 香港ﾠ`”
*   **错误**：“`英国`”（代理组没有带识别标识符，会被系统消除！）

5. 在“引入代理”中添加对应的国家/地区的节点（例如：英国代理组添加英国节点）。

6. 点击右下角**保存**。

> **附录\***：在“代理组类型中”选择“根据 URL 测试延迟选择代理”（根据该节点网络延迟自动选择节点）或者“不可用时切换到另一个代理”（一个节点不通则自动连接另一个节点）可以实现该国家/地区中的自动节点切换。

### 第三步：重新加载，部署完成
1. 设定好脚本并建好带有标识符的组后，再次进入左侧“订阅”界面。
2. 右键点击你的订阅，选择“重新加载”或手动点一下更新。
3. 部署完成，切回主页看看吧！

---

## 使用指南：
点击“代理组”的“**🚀 区域切换**”，“节点”选择你想要连接的国家/地区。
“代理组”选择你想连接的地区，“节点”选择你想要连接的目标节点，即可连接。

如果你想要实现某个地区的节点自动切换，可以参考“附录*”，即可实现该地区自动切换节点，这样你就可以只在“代理组”的“🚀 区域切换”选择地区就可以轻松实现目标国家/地区切换。

---

## 问题排查：
*   **当想要使用外部节点访问国内网站时**：在“代理组”中选择“🇨🇳 国内直连”，将其节点切换为“🚀 区域切换”，则可使其通过外部节点访问国内网站。
*   **当网页打不开想要排查是否是由于拦截广告导致网页访问拒绝时**：在“代理组”中选择“🚫 广告拦截”，将其节点切换为“🚀 区域切换”或者“DIRECT”（直连）即可排查问题。
