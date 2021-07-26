title: SwitchyOmega 备份
date: '2021-06-22 20:39:55'
updated: '2021-06-22 20:40:25'
category: 
 - 代理
tags: [SwitchyOmega, proxy, 备份]
permalink: /articles/2021/06/22/1624365595584.html
---
`OmegaProfile_auto_switch.pac` ：

```javascript
var FindProxyForURL = function(init, profiles) {
    return function(url, host) {
        "use strict";
        var result = init, scheme = url.substr(0, url.indexOf(":"));
        do {
            result = profiles[result];
            if (typeof result === "function") result = result(url, host, scheme);
        } while (typeof result !== "string" || result.charCodeAt(0) === 43);
        return result;
    };
}("+auto switch", {
    "+auto switch": function(url, host, scheme) {
        "use strict";
        if (/(?:^|\.)acgur\.com$/.test(host)) return "+proxy";
        if (/(?:^|\.)golang\.org$/.test(host)) return "+proxy";
        if (/(?:^|\.)githubassets\.com$/.test(host)) return "+proxy";
        if (/(?:^|\.)cloudflare\.com$/.test(host)) return "+proxy";
        if (/(?:^|\.)apache\.org$/.test(host)) return "+proxy";
        if (/(?:^|\.)93dd4aa065069a75\.com$/.test(host)) return "+proxy";
        if (/(?:^|\.)camo\.githubusercontent\.com$/.test(host)) return "+proxy";
        if (/(?:^|\.)avatars\.githubusercontent\.com$/.test(host)) return "+proxy";
        if (/(?:^|\.)jetbrains\.com$/.test(host)) return "+proxy";
        if (/(?:^|\.)mvnrepository\.com$/.test(host)) return "+proxy";
        if (/(?:^|\.)twimg\.com$/.test(host)) return "+proxy";
        if (/(?:^|\.)twitter\.com$/.test(host)) return "+proxy";
        if (/(?:^|\.)chromium\.org$/.test(host)) return "+proxy";
        if (/(?:^|\.)nyaa\.si$/.test(host)) return "+proxy";
        if (/(?:^|\.)adsco\.re$/.test(host)) return "+proxy";
        if (/(?:^|\.)advinci\.uno$/.test(host)) return "+proxy";
        if (/(?:^|\.)bootstrapcdn\.com$/.test(host)) return "+proxy";
        if (/(?:^|\.)blogger\.com$/.test(host)) return "+proxy";
        if (/(?:^|\.)gstatic\.com$/.test(host)) return "+proxy";
        if (/(?:^|\.)bp\.blogspot\.com$/.test(host)) return "+proxy";
        if (/(?:^|\.)javx01\.blogspot\.com$/.test(host)) return "+proxy";
        if (/(?:^|\.)eyny\.com$/.test(host)) return "+proxy";
        if (/(?:^|\.)cnxalex\.blogspot\.com$/.test(host)) return "+proxy";
        if (/(?:^|\.)zimuku\.la$/.test(host)) return "+proxy";
        if (/(?:^|\.)reimu\.net$/.test(host)) return "+proxy";
        if (/(?:^|\.)novaopcj\.top$/.test(host)) return "+proxy";
        if (/(?:^|\.)optimizely\.com$/.test(host)) return "+proxy";
        if (/(?:^|\.)rawgit\.com$/.test(host)) return "+proxy";
        if (/(?:^|\.)gtimg\.com$/.test(host)) return "DIRECT";
        if (/(?:^|\.)qq\.com$/.test(host)) return "DIRECT";
        if (/(?:^|\.)telegram\.org$/.test(host)) return "+proxy";
        if (/(?:^|\.)v2ray\.com$/.test(host)) return "+proxy";
        if (/(?:^|\.)apkfab\.com$/.test(host)) return "+proxy";
        if (/(?:^|\.)wikipedia\.org$/.test(host)) return "+proxy";
        if (/(?:^|\.)raw\.githubusercontent\.com$/.test(host)) return "+proxy";
        if (/(?:^|\.)vercel\.com$/.test(host)) return "+proxy";
        if (/(?:^|\.)telegra\.ph$/.test(host)) return "+proxy";
        if (/(?:^|\.)googlevideo\.com$/.test(host)) return "+proxy";
        if (/(?:^|\.)doubleclick\.net$/.test(host)) return "+proxy";
        if (/(?:^|\.)ytimg\.com$/.test(host)) return "+proxy";
        if (/(?:^|\.)ggpht\.com$/.test(host)) return "+proxy";
        if (/(?:^|\.)youtube\.com$/.test(host)) return "+proxy";
        if (/(?:^|\.)facebook\.com$/.test(host)) return "+proxy";
        if (/(?:^|\.)sstatic\.net$/.test(host)) return "+proxy";
        if (/(?:^|\.)stackoverflow\.com$/.test(host)) return "+proxy";
        if (/(?:^|\.)imgur\.com$/.test(host)) return "+proxy";
        if (/(?:^|\.)biuaxia\.cn$/.test(host)) return "DIRECT";
        if (/(?:^|\.)github\.com$/.test(host)) return "+proxy";
        if (/(?:^|\.)tencent-cloud\.com$/.test(host)) return "DIRECT";
        if (/(?:^|\.)ajax\.googleapis\.com$/.test(host)) return "+proxy";
        if (/(?:^|\.)v2ex\.com$/.test(host)) return "+proxy";
        if (/(?:^|\.)microsoft\.com$/.test(host)) return "+proxy";
        if (/(?:^|\.)atlassian\.net$/.test(host)) return "+proxy";
        if (/(?:^|\.)netsarang\.com$/.test(host)) return "+proxy";
        if (/(?:^|\.)google\.com$/.test(host)) return "+proxy";
        if (/(?:^|\.)googleusercontent\.com$/.test(host)) return "+proxy";
        if (/(?:^|\.)gravatar\.com$/.test(host)) return "+proxy";
        if (/(?:^|\.)t\.me$/.test(host)) return "+proxy";
        if (/^apkfab\.com$/.test(host)) return "+proxy";
        if (/(?:)/.test(host)) return "+proxy";
        return "DIRECT";
    },
    "+proxy": function(url, host, scheme) {
        "use strict";
        if (/^127\.0\.0\.1$/.test(host) || /^::1$/.test(host) || /^localhost$/.test(host)) return "DIRECT";
        return "PROXY localhost:8889";
    }
});
```

编辑源代码：

```javascript
[SwitchyOmega Conditions]
@with result

*.acgur.com +proxy
*.golang.org +proxy
*.githubassets.com +proxy
*.cloudflare.com +proxy
*.apache.org +proxy
*.93dd4aa065069a75.com +proxy
*.camo.githubusercontent.com +proxy
*.avatars.githubusercontent.com +proxy
*.jetbrains.com +proxy
*.mvnrepository.com +proxy
*.twimg.com +proxy
*.twitter.com +proxy
*.chromium.org +proxy
*.nyaa.si +proxy
*.adsco.re +proxy
*.advinci.uno +proxy
*.bootstrapcdn.com +proxy
*.blogger.com +proxy
*.gstatic.com +proxy
*.bp.blogspot.com +proxy
*.javx01.blogspot.com +proxy
*.eyny.com +proxy
*.cnxalex.blogspot.com +proxy
*.zimuku.la +proxy
*.reimu.net +proxy
*.novaopcj.top +proxy
*.optimizely.com +proxy
*.rawgit.com +proxy
*.gtimg.com +direct
*.qq.com +direct
*.telegram.org +proxy
*.v2ray.com +proxy
*.apkfab.com +proxy
*.wikipedia.org +proxy
*.raw.githubusercontent.com +proxy
*.vercel.com +proxy
*.telegra.ph +proxy
*.googlevideo.com +proxy
*.doubleclick.net +proxy
*.ytimg.com +proxy
*.ggpht.com +proxy
*.youtube.com +proxy
*.facebook.com +proxy
*.sstatic.net +proxy
*.stackoverflow.com +proxy
*.imgur.com +proxy
*.biuaxia.cn +direct
*.github.com +proxy
*.tencent-cloud.com +direct
*.ajax.googleapis.com +proxy
*.v2ex.com +proxy
211.83.241.125 +direct
*.microsoft.com +proxy
*.atlassian.net +proxy
*.netsarang.com +proxy
*.google.com +proxy
*.googleusercontent.com +proxy
*.gravatar.com +proxy
*.t.me +proxy
apkfab.com +proxy

* +direct

```
