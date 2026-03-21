function main(config) {

  // 直连代理组名单（请在此添加原配置中的直连代理组名称）
  const directList = ["🌏 国内网站", "CN"]; 
  
  // 广告拦截组名单（若原配置有去广告组，请在此添加原配置中的广告拦截组名称）
  const adblockList = ["广告拦截", "AdBlock"];
  
  // 标识符 (默认为空白符“U+FFA0”)
  // 软件内添加代理组后，请精准复制括号内的隐藏字符（ﾠ），粘贴到代理组名称的尾部，例如：“🇨🇳 香港ﾠ”
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
