---
title: Hacklandray
date: 2021-12-23 11:12:12
comment: false
toc: true
theme: xray
password: bourne
---

## 破解后的效果

![Snipaste20211223112124.png](https://b3logfile.com/file/2021/12/Snipaste_2021-12-23_11-21-24-cbf92444.png)

![Snipaste20211223112301.png](https://b3logfile.com/file/2021/12/Snipaste_2021-12-23_11-23-01-a76398b0.png)

## 破解弹框提示

移除：

1. 您使用的蓝凌软件产品许可将于 `xx` 到期，到期后系统将不能使用，同时部署套数已超出许可限制，当前系统将于 `xx` 自动停止，请尽快联系系统管理员解决！
2. 您使用的蓝凌软件产品许可将于 `xx` 到期，到期后系统将不能使用，同时由于网络原因无法连接蓝凌许可验证服务器，当前系统将于 `xx` 自动停止，请尽快联系系统管理员解决！
3. 您使用的蓝凌软件产品部署套数已超出许可限制，当前系统将于 `xx` 自动停止，请尽快联系系统管理员解决！
4. 您使用的蓝凌软件产品许可由于网络原因无法连接蓝凌许可验证服务器，当前系统将于 `xx` 自动停止，请尽快联系系统管理员解决！
5. 您使用的蓝凌软件产品许可将于 `xx` 到期，到期后系统将不能使用，同时部署套数已超出许可限制，当前系统将于 `xx` 自动停止，请尽快联系蓝凌项目组更换许可，或者拨打蓝凌服务热线：4006-222-312 转2号线
6. 您使用的蓝凌软件产品许可将于 `xx` 到期，到期后系统将不能使用，同时由于网络原因无法连接蓝凌许可验证服务器，当前系统将于 `xx` 自动停止，请尽快联系蓝凌项目组更换许可，或者拨打蓝凌服务热线：4006-222-312 转2号线
7. 您使用的蓝凌软件产品将于 `xx` 到期，到期后系统将不能使用，请尽快联系蓝凌项目组更换许可，或者拨打蓝凌服务热线：4006-222-312 转2号线
8. 您使用的蓝凌软件产品部署套数已超出许可限制，当前系统将于 `xx` 自动停止，请尽快联系蓝凌项目组更换许可，或者拨打蓝凌服务热线：4006-222-312 转2号线
9. 您使用的蓝凌软件产品许可由于网络原因无法连接蓝凌许可验证服务器，当前系统将于 `xx` 自动停止，请尽快联系蓝凌项目组更换许可，或者拨打蓝凌服务热线：4006-222-312 转2号线

注意 `com/landray/kmss/sys/portal/taglib/HeaderTag.java:89`，代码如下：

```java
package com.landray.kmss.sys.portal.taglib;

import com.landray.kmss.sys.config.util.LicenseUtil;
import com.landray.kmss.sys.portal.util.PortalUtil;
import com.landray.kmss.sys.portal.util.SysPortalInfo;
import com.landray.kmss.sys.portal.xml.model.SysPortalHeader;
import com.landray.kmss.sys.ui.taglib.template.TargetUrlContentAcquirer;
import com.landray.kmss.sys.ui.taglib.template.AbstractTemplateTag.ParamManager;
import com.landray.kmss.util.StringUtil;
import java.io.IOException;
import java.net.URLDecoder;
import java.util.*;
import java.util.Map.Entry;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpSession;
import javax.servlet.jsp.JspException;
import javax.servlet.jsp.JspTagException;
import javax.servlet.jsp.PageContext;
import javax.servlet.jsp.tagext.DynamicAttributes;
import javax.servlet.jsp.tagext.SimpleTagSupport;
import net.sf.json.JSONObject;
import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;

public class HeaderTag extends SimpleTagSupport implements DynamicAttributes {
    private static final Log logger = LogFactory.getLog(HeaderTag.class);
    private ParamManager params;
    private static final String charEncoding = "UTF-8";
    private String ref;
    private String scene;
    protected Map<String, String> vars;

    public HeaderTag() {
    }

    public String getRef() {
        return this.ref;
    }

    public void setRef(String ref) {
        this.ref = ref;
    }

    public String getScene() {
        return this.scene;
    }

    public void setScene(String scene) {
        this.scene = scene;
    }

    public void doTag() throws JspException {
        PageContext pageContext = (PageContext)this.getJspContext();
        this.licenseNotify(pageContext);
        this.outputHeaderHtml(pageContext);
        if (pageContext.getRequest().getParameter("designertime") != null && pageContext.getRequest().getParameter("designertime").equals("yes")) {
            try {
                pageContext.getOut().print("<div data-lui-mark='template:header' key='headerDesigner'></div>");
            } catch (IOException var7) {
                logger.error(var7);
            }
        } else {
            String file = this.getFile();
            if (StringUtil.isNotNull(file)) {
                String targetUrl = TargetUrlContentAcquirer.coverUrl(file, pageContext);
                String value = (new TargetUrlContentAcquirer(targetUrl, pageContext, "UTF-8")).acquireString();

                try {
                    value = "<div class='lui_portal_header' style='display:none;'>" + value + "</div>";
                    value = value + "<script>seajs.use(['lui/jquery','theme!portal'],function($){$('.lui_portal_header').show();});</script>";
                    pageContext.getOut().print(value);
                } catch (IOException var6) {
                    logger.error(var6);
                }
            } else if (!"anonymous".equals(this.scene)) {
                this.outputPortalNotice(pageContext);
            }
        }

        this.ref = null;
        this.scene = null;
        this.params = null;
    }

    private void licenseNotify(PageContext pageContext) {
        try {
            HttpSession session = pageContext.getSession();
            String notify = (String)session.getAttribute("KMSS_LICENSE_NOTIFY");
            // if (notify != null) {
            if (true) {
                System.out.println("LicenseUtil getMap方法打印:" + LicenseUtil.getMap());
                return;
            }

            session.setAttribute("KMSS_LICENSE_NOTIFY", "TRUE");
            String subject = (String)session.getAttribute("KMSS_NOTIFY_REGIST_OVERFLOW");
            if (subject != null) {
                subject = URLDecoder.decode(subject, "UTF-8");
                session.removeAttribute("KMSS_NOTIFY_REGIST_OVERFLOW");
            }

            subject = StringUtil.linkString(LicenseUtil.getExpireSubject(2), "\\r\\n", subject);
            if (subject == null) {
                return;
            }

            pageContext.getOut().write("<script>alert('" + StringUtil.replace(subject, "'", "\\'") + "');</script>");
        } catch (IOException var5) {
            logger.error(var5);
        }

    }

    private void outputHeaderHtml(PageContext pageContext) {
        try {
            String html = (String)pageContext.getRequest().getAttribute("SYS_PORTAL_HEADER_HTML");
            if (StringUtil.isNotNull(html)) {
                pageContext.getOut().write(html);
            }
        } catch (IOException var3) {
            logger.error(var3);
        }

    }

    protected String getRefFile() throws JspTagException {
        String file = ((SysPortalHeader)PortalUtil.getPortalHeaders().get(this.ref)).getFdFile();
        if (this.params == null) {
            this.params = new ParamManager();
        }

        Iterator iter = this.getVars().entrySet().iterator();

        while(iter.hasNext()) {
            Entry<String, String> entry = (Entry)iter.next();
            this.params.addParameter((String)entry.getKey(), (String)entry.getValue());
        }

        if (this.params != null) {
            if (StringUtil.isNotNull(this.getScene())) {
                this.params.addParameter("scene", this.getScene());
            }

            file = this.params.aggregateParams(file, "UTF-8");
        }

        return file;
    }

    protected String getFile() throws JspTagException {
        try {
            if (StringUtil.isNotNull(this.ref)) {
                return this.getRefFile();
            }

            PageContext ctx = (PageContext)this.getJspContext();
            SysPortalInfo info = PortalUtil.getPortalInfo((HttpServletRequest)ctx.getRequest());
            String header = info.getHeaderId();
            if (StringUtil.isNotNull(header)) {
                String file = ((SysPortalHeader)PortalUtil.getPortalHeaders().get(header)).getFdFile();
                List<String> names = new ArrayList();
                if (this.getVars().size() > 0) {
                    if (this.params == null) {
                        this.params = new ParamManager();
                    }

                    Iterator iter = this.getVars().entrySet().iterator();

                    while(iter.hasNext()) {
                        Entry<String, String> entry = (Entry)iter.next();
                        names.add((String)entry.getKey());
                        this.params.addParameter((String)entry.getKey(), (String)entry.getValue());
                    }
                }

                if (StringUtil.isNotNull(info.getHeaderVars())) {
                    JSONObject json = JSONObject.fromObject(info.getHeaderVars());
                    if (json != null && !json.isNullObject()) {
                        if (this.params == null) {
                            this.params = new ParamManager();
                        }

                        Iterator keys = json.keys();

                        while(keys.hasNext()) {
                            String key = keys.next().toString();
                            if (!names.contains(key)) {
                                this.params.addParameter(key, json.getString(key));
                            }
                        }
                    }
                }

                if (this.params != null) {
                    if (StringUtil.isNotNull(this.getScene())) {
                        this.params.addParameter("scene", this.getScene());
                    }

                    file = this.params.aggregateParams(file, "UTF-8");
                }

                return file;
            }
        } catch (Exception var9) {
            logger.error(var9);
        }

        return null;
    }

    public Map<String, String> getVars() {
        if (this.vars == null) {
            this.vars = new HashMap();
        }

        return this.vars;
    }

    public void setDynamicAttribute(String uri, String key, Object value) throws JspException {
        if (key.startsWith("var-")) {
            this.getVars().put(key.substring(4), value.toString());
        }

    }

    private void outputPortalNotice(PageContext pageContext) {
        try {
            String noticeUrl = PortalUtil.getPortalNoticeUrl();
            String value = (new TargetUrlContentAcquirer(noticeUrl, pageContext, "UTF-8")).acquireString();
            pageContext.getOut().print(value);
        } catch (Exception var4) {
            logger.error(var4);
        }

    }
}
```

还可以直接注释掉 `com/landray/kmss/sys/portal/taglib/HeaderTag.java:107`，就不会提示了。

## 破解 License 信息

## 基本信息

代码如下：

```java
package com.landray.kmss.sys.config.util;

import com.landray.kmss.sys.appconfig.model.SysAppConfig;
import com.landray.kmss.sys.config.service.ISysDaemonService;
import com.landray.kmss.util.DateUtil;
import com.landray.kmss.util.ResourceUtil;
import com.landray.kmss.util.SpringBeanUtil;
import com.landray.kmss.util.StringUtil;

import java.net.InetAddress;
import java.net.NetworkInterface;
import java.util.*;

public class LicenseUtil {
    public static final int NOTIFY_SYS_JOB = 1;
    public static final int NOTIFY_SYS_BROWSER = 2;
    private static Map<String, String> parameters = new HashMap();
    private static final String CONFIG_KEY = new String(new byte[]{99, 111, 109, 46, 108, 97, 110, 100, 114, 97, 121, 46, 107, 109, 115, 115, 46, 115, 121, 115, 46, 99, 111, 110, 102, 105, 103, 46, 109, 111, 100, 101, 108, 46, 83, 121, 115, 68, 97, 101, 109, 111, 110});
    private static final String CONFIG_FIELD = new String(new byte[]{116, 97, 105, 108});
    private static final String CONFIG_EXPFIELD = new String(new byte[]{110, 101, 116, 101, 120, 112, 105, 114, 101});

    // 软件授权名称
    public static final String TO_KEY = new String(new byte[]{108, 105, 99, 101, 110, 115, 101, 45, 116, 111});
    public static final String TO_KEY_TARGET = new String(new byte[]{-23, -109, -126, -26, -127, -87, -25, -89, -111, -26, -118, -128});
    // 正式版
    public static final String TYPE_KEY = new String(new byte[]{108, 105, 99, 101, 110, 115, 101, 45, 116, 121, 112, 101});
    public static final String TYPE_KEY_TARGET = new String(new byte[]{79, 102, 102, 105, 99, 105, 97, 108});
    // 授权人数
    public static final String ORG_PERSON_KEY = new String(new byte[]{108, 105, 99, 101, 110, 115, 101, 45, 111, 114, 103, 45, 112, 101, 114, 115, 111, 110});
    public static final String ORG_PERSON_KEY_TARGET = new String(new byte[]{57, 57, 57, 57, 57, 57, 57});
    // 群集数
    public static final String CLUSTER_KEY = new String(new byte[]{108, 105, 99, 101, 110, 115, 101, 45, 99, 108, 117, 115, 116, 101, 114});
    public static final String CLUSTER_KEY_TARGET = new String(new byte[]{45, 49});
    // 标题
    public static final String TITLE_KEY = new String(new byte[]{108, 105, 99, 101, 110, 115, 101, 45, 116, 105, 116, 108, 101});
    public static final String TITLE_KEY_TARGET = new String(new byte[]{});
    // 版权
    public static final String HOME_KEY = new String(new byte[]{108, 105, 99, 101, 110, 115, 101, 45, 104, 111, 109, 101});
    public static final String HOME_KEY_TARGET = new String(new byte[]{-23, -109, -126, -26, -127, -87, -25, -119, -120, -26, -99, -125, -26, -119, -128, -26, -100, -119});
    // 手机在线限制
    public static final String PHONE_KEY = new String(new byte[]{108, 105, 99, 101, 110, 115, 101, 45, 112, 100, 97, 45, 112, 104, 111, 110, 101});
    public static final String PHONE_KEY_TARGET = new String(new byte[]{57, 57, 57, 57, 57, 57, 57});
    public static final String I_PHONE_KEY = new String(new byte[]{108, 105, 99, 101, 110, 115, 101, 45, 112, 100, 97, 45, 112, 104, 111, 110, 101});
    public static final String I_PHONE_KEY_TARGET = new String(new byte[]{57, 57, 57, 57, 57, 57, 57});
    // 在线数
    public static final String ONLINE_KEY = new String(new byte[]{108, 105, 99, 101, 110, 115, 101, 45, 112, 100, 97, 45, 112, 104, 111, 110, 101});
    public static final String ONLINE_KEY_TARGET = new String(new byte[]{57, 57, 57, 57, 57, 57, 57});

    public LicenseUtil() {
    }

    public static Map<String, String> getMap() {
        return parameters;
    }

    public static String get(String key) {
        return parseKey(key);
    }

    private static String parseKey(String key) {
        if (key.equals(TO_KEY)) {
            return TO_KEY_TARGET;
        } else if (key.equals(TYPE_KEY)) {
            return TYPE_KEY_TARGET;
        } else if (key.equals(ORG_PERSON_KEY)) {
            return ORG_PERSON_KEY_TARGET;
        } else if (key.equals(CLUSTER_KEY)) {
            return CLUSTER_KEY_TARGET;
        } else if (key.equals(TITLE_KEY)) {
            return TITLE_KEY_TARGET;
        } else if (key.equals(HOME_KEY)) {
            return HOME_KEY_TARGET;
        } else if (key.equals(PHONE_KEY)) {
            return PHONE_KEY_TARGET;
        } else if (key.equals(I_PHONE_KEY)) {
            return I_PHONE_KEY_TARGET;
        } else if (key.equals(ONLINE_KEY)) {
            return ONLINE_KEY_TARGET;
        } else {
            return parameters.get(key);
        }
    }

    public static void set(Map<String, String> parameters) {
        StackTraceElement[] stackTrace = Thread.currentThread().getStackTrace();
        System.out.println("LicenseUtil 方法调用链打印:" + Arrays.toString(stackTrace));
        LicenseUtil.parameters = parameters;
    }

    public static String getExpireSubject(int type) {
        long now = System.currentTimeMillis();
        String licenseType = get("license-type");
        if ("Official".equals(licenseType)) {
            return null;
        } else {
            String reason1 = null;
            String reason2 = null;
            String reason3 = null;
            String licenseExpire = get("license-expire");
            long day = (now - DateUtil.convertStringToDate(licenseExpire, "yyyy-MM-dd").getTime()) / 86400000L;
            String[] calculateNetReason;
            if (type == 2) {
                if (day <= -7L) {
                    reason1 = null;
                } else {
                    reason1 = licenseExpire;
                }

                calculateNetReason = calculateNetReason(2);
                reason2 = calculateNetReason[0];
                reason3 = calculateNetReason[1];
                if (reason1 != null) {
                    if (reason2 != null) {
                        return "您使用的蓝凌软件产品许可将于" + reason1 + "到期，到期后系统将不能使用，同时部署套数已超出许可限制，当前系统将于" + reason2 + "自动停止，请尽快联系系统管理员解决！";
                    } else {
                        return reason3 != null ? "您使用的蓝凌软件产品许可将于" + reason1 + "到期，到期后系统将不能使用，同时由于网络原因无法连接蓝凌许可验证服务器，当前系统将于" + reason3 + "自动停止，请尽快联系系统管理员解决！" : "您使用的蓝凌软件产品将于" + reason1 + "到期，到期后系统将不能使用，请尽快联系系统管理员解决！";
                    }
                } else if (reason2 != null) {
                    return "您使用的蓝凌软件产品部署套数已超出许可限制，当前系统将于" + reason2 + "自动停止，请尽快联系系统管理员解决！";
                } else {
                    return reason3 != null ? "您使用的蓝凌软件产品许可由于网络原因无法连接蓝凌许可验证服务器，当前系统将于" + reason3 + "自动停止，请尽快联系系统管理员解决！" : null;
                }
            } else {
                if (day >= -30L && (day > -7L || day % 7L == 0L)) {
                    reason1 = licenseExpire;
                } else {
                    reason1 = null;
                }

                calculateNetReason = calculateNetReason(1);
                reason2 = calculateNetReason[0];
                reason3 = calculateNetReason[1];
                if (reason1 != null) {
                    if (reason2 != null) {
                        return "您使用的蓝凌软件产品许可将于" + reason1 + "到期，到期后系统将不能使用，同时部署套数已超出许可限制，当前系统将于" + reason2 + "自动停止，请尽快联系蓝凌项目组更换许可，或者拨打蓝凌服务热线：4006-222-312 转2号线";
                    } else {
                        return reason3 != null ? "您使用的蓝凌软件产品许可将于" + reason1 + "到期，到期后系统将不能使用，同时由于网络原因无法连接蓝凌许可验证服务器，当前系统将于" + reason3 + "自动停止，请尽快联系蓝凌项目组更换许可，或者拨打蓝凌服务热线：4006-222-312 转2号线" : "您使用的蓝凌软件产品将于" + reason1 + "到期，到期后系统将不能使用，请尽快联系蓝凌项目组更换许可，或者拨打蓝凌服务热线：4006-222-312 转2号线";
                    }
                } else if (reason2 != null) {
                    return "您使用的蓝凌软件产品部署套数已超出许可限制，当前系统将于" + reason2 + "自动停止，请尽快联系蓝凌项目组更换许可，或者拨打蓝凌服务热线：4006-222-312 转2号线";
                } else {
                    return reason3 != null ? "您使用的蓝凌软件产品许可由于网络原因无法连接蓝凌许可验证服务器，当前系统将于" + reason3 + "自动停止，请尽快联系蓝凌项目组更换许可，或者拨打蓝凌服务热线：4006-222-312 转2号线" : null;
                }
            }
        }
    }

    private static String[] calculateNetReason(int type) {
        String[] ret = new String[2];
        if (isEnableRemoteCheck() && enablePermitNum()) {
            SysAppConfig config = loadConfig(CONFIG_KEY + "." + CONFIG_FIELD, CONFIG_EXPFIELD);
            if (config != null) {
                String value = config.getFdValue();
                if (StringUtil.isNotNull(value)) {
                    String[] split = value.split(";");
                    long expireTime = Long.parseLong(split[1]);
                    long now = System.currentTimeMillis();
                    long day = (now - expireTime) / 86400000L;
                    if (type == 2) {
                        if ("0".equals(split[0])) {
                            ret[1] = null;
                            if (day <= -7L) {
                                ret[0] = null;
                            } else {
                                ret[0] = DateUtil.convertDateToString(new Date(expireTime), "yyyy-MM-dd");
                            }
                        } else {
                            ret[0] = null;
                            if (day <= -7L) {
                                ret[1] = null;
                            } else {
                                ret[1] = DateUtil.convertDateToString(new Date(expireTime), "yyyy-MM-dd");
                            }
                        }
                    } else if ("0".equals(split[0])) {
                        ret[1] = null;
                        if (day >= -30L && (day > -7L || day % 7L == 0L)) {
                            ret[0] = DateUtil.convertDateToString(new Date(expireTime), "yyyy-MM-dd");
                        } else {
                            ret[0] = null;
                        }
                    } else {
                        ret[0] = null;
                        if (day >= -30L && (day > -7L || day % 7L == 0L)) {
                            ret[1] = DateUtil.convertDateToString(new Date(expireTime), "yyyy-MM-dd");
                        } else {
                            ret[1] = null;
                        }
                    }
                }
            }
        }

        return ret;
    }

    private static SysAppConfig loadConfig(String key, String field) {
        List<?> list = getDaemonService().getHibernateTemplate().find("from SysAppConfig where fdKey=? and fdField=? order by fdId", new Object[]{key, field});
        return list.isEmpty() ? null : (SysAppConfig) list.get(0);
    }

    private static ISysDaemonService getDaemonService() {
        return (ISysDaemonService) SpringBeanUtil.getBean("sysDaemonService");
    }

    public static boolean isEnableRemoteCheck() {
        return "true".equalsIgnoreCase(get("license-information-collection"));
    }

    public static boolean enablePermitNum() {
        String string = get(new String(new byte[]{108, 105, 99, 101, 110, 115, 101, 45, 112, 101, 114, 109, 105, 116, 45, 110, 117, 109}));

        try {
            int num = Integer.parseInt(string);
            if (num > 0) {
                return true;
            }
        } catch (Exception var2) {
        }

        return false;
    }

    public static String getLink() {
        return "http://www.myekp.net";
    }

    public static Map<String, String> getServerHwaddr() {
        Map<String, String> serverHwaddr = new HashMap();
        String jip = System.getProperty("jgroups.bind_addr");
        if (jip != null) {
            jip = jip.trim();
            if (jip.length() == 0) {
                jip = null;
            }
        }

        try {
            if (jip != null) {
                Enumeration<NetworkInterface> nets = NetworkInterface.getNetworkInterfaces();
                StringBuffer mac = new StringBuffer();
                boolean found = false;

                int i;
                label120:
                while (true) {
                    byte[] orimac;
                    do {
                        do {
                            NetworkInterface net;
                            do {
                                do {
                                    do {
                                        if (!nets.hasMoreElements()) {
                                            break label120;
                                        }

                                        net = (NetworkInterface) nets.nextElement();
                                    } while (net.isLoopback());
                                } while (!net.isUp());

                                Enumeration addrs = net.getInetAddresses();

                                while (addrs.hasMoreElements()) {
                                    InetAddress addr = (InetAddress) addrs.nextElement();
                                    if (jip.equalsIgnoreCase(addr.getHostAddress())) {
                                        found = true;
                                        break;
                                    }
                                }
                            } while (!found);

                            orimac = net.getHardwareAddress();
                        } while (orimac == null);
                    } while (orimac.length == 0);

                    for (i = 0; i < orimac.length; ++i) {
                        if (i != 0) {
                            mac.append(":");
                        }

                        int temp = orimac[i] & 255;
                        String str = Integer.toHexString(temp);
                        if (str.length() == 1) {
                            mac.append("0" + str);
                        } else {
                            mac.append(str);
                        }
                    }

                    if (found) {
                        serverHwaddr.put("ip", jip);
                        serverHwaddr.put("mac", mac.toString().toUpperCase());
                        break;
                    }
                }

                if (!found) {
                    InetAddress addr = InetAddress.getLocalHost();
                    byte[] orimac = NetworkInterface.getByInetAddress(addr).getHardwareAddress();
                    mac = new StringBuffer("");

                    for (i = 0; i < orimac.length; ++i) {
                        if (i != 0) {
                            mac.append(":");
                        }

                        i = orimac[i] & 255;
                        String str = Integer.toHexString(i);
                        if (str.length() == 1) {
                            mac.append("0" + str);
                        } else {
                            mac.append(str);
                        }
                    }

                    serverHwaddr.put("error", ResourceUtil.getString("license.address.mismatch", (String) null, (Locale) null, new Object[]{addr.getHostAddress(), mac.toString().toUpperCase(), jip}));
                }
            } else {
                InetAddress addr = InetAddress.getLocalHost();
                byte[] orimac = NetworkInterface.getByInetAddress(addr).getHardwareAddress();
                StringBuffer mac = new StringBuffer("");

                for (int i = 0; i < orimac.length; ++i) {
                    if (i != 0) {
                        mac.append(":");
                    }

                    int temp = orimac[i] & 255;
                    String str = Integer.toHexString(temp);
                    if (str.length() == 1) {
                        mac.append("0" + str);
                    } else {
                        mac.append(str);
                    }
                }

                serverHwaddr.put("ip", addr.getHostAddress());
                serverHwaddr.put("mac", mac.toString().toUpperCase());
            }
        } catch (Exception var11) {
            serverHwaddr.put("error", ResourceUtil.getString("license.address.failed"));
        }

        return serverHwaddr;
    }
}
```

注意 `parseKey` 方法的判断和 `getMap` 可以实现更多自定义功能！

### 系统概览界面的数据伪装

如图：

![Snipaste20211223115716.png](https://b3logfile.com/file/2021/12/Snipaste_2021-12-23_11-57-16-7af147b4.png)

其他数据也可以通过伪装实现，但已经在 [基本信息](#基本信息) 做过修改了。

### 其它调整


1. 表 `sys_config_data` 中 `fd_key` 为 `` 调整为允许的在线数（待验证）

  ![Snipaste20211223115917.png](https://b3logfile.com/file/2021/12/Snipaste_2021-12-23_11-59-17-a740bc26.png)

