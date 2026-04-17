<%*
//-------------------------------------------
// 功能:
// 1. 根据文件夹领域(area)创建笔记
// 命名规则: "领域-笔记主题"
//-------------------------------------------

const DEFAULT_AREA_CHOISE_OPTION = "继承领域";
const areaBasedOptions = ["无领域(默认)","继承领域"];
const areaBased = await tp.system.suggester(areaBasedOptions, areaBasedOptions, false, "创建模式");
const finalAreaBased = areaBased || "无领域(默认)";

// 确认笔记标题
let originaltitle = tp.file.title.trim();
let title = originaltitle;
const defaultNames = ["未命名", "untitled", "无标题"];
if (!originaltitle || defaultNames.includes(originaltitle.toLowerCase())) {
     title= await tp.system.prompt("请输入笔记主题:", "", true);
} else if (originaltitle.includes("-")) {
        title = originaltitle.split("-").slice(0, -1).join("-");
}

// 变更笔记文件名
let areaType = null;
let newFileName = title;
if (finalAreaBased === DEFAULT_AREA_CHOISE_OPTION) {
    try {
        const folderFrontMatter = await tp.user.utils.getFolderNoteFrontMatter(tp);
        if (folderFrontMatter) {
            areaType = folderFrontMatter["domain"];
        }
		newFileName = areaType ? `${areaType}-${title}` : title;
    } catch (e) {
        console.debug("获取文件夹元数据失败:", e);
    }
}

if (originaltitle != newFileName){
	newFileName = areaType ? `${areaType}-${title}` : title;
	await tp.file.rename(newFileName);
}

try {
    // 配置 Frontmatter
    let yaml = `---\n`;

    // 别名信息
    yaml += `aliases:\n`;
    if (areaType) yaml += `  - "abbr://${areaType}/${title}"\n`;
    if (title != newFileName) yaml += `  - "${title}"\n`;
	//console.debug(`title:${title}, newFileName:${newFileName}	`);
    // 其他元信息
    yaml += `tags:\n`;
	yaml += `- \n`;
    yaml += `title: "${title}"\n`;
    yaml += `description: \n`;
    yaml += `domain: "${title}"\n`;
    yaml += `created: ${tp.date.now("YYYY-MM-DD HH:mm")}\n`;
    yaml += `---\n`;

    tR += yaml;

    // 动态生成正文内容结构
    const content = `\n## 内容\n- \n`;
    tR += content;

} catch (e) {
    console.error("生成 YAML 时出错:", e);
}
%>