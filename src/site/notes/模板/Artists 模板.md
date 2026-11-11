---
{"dg-publish":true,"permalink":"/模板/Artists 模板/","created":"2026-10-28T16:37:01.997+08:00","dg-note-properties":{}}
---


<div class="artist-card"><div class="artist-card__left"><div class="artist-card__image-wrap"></div><h2 class="artist-card__name">Artists 模板</h2><div class="artist-card__facts"><div class="artist-card__facts-top"></div></div></div></div>

<p><span></span></p>



<pre class="dataview dataview-error">Evaluation Error: SyntaxError: Invalid regular expression: missing /
    at DataviewInlineApi.&lt;anonymous&gt; (plugin:dataview:19027:21)
    at evalInContext (plugin:dataview:19028:7)
    at asyncEvalInContext (plugin:dataview:19038:32)
    at DataviewJSRenderer.render (plugin:dataview:19064:19)
    at DataviewJSRenderer.onload (plugin:dataview:18606:14)
    at DataviewJSRenderer.load (app://obsidian.md/app.js:205:11318)
    at DataviewApi.executeJs (plugin:dataview:19607:18)
    at DataviewCompiler.&lt;anonymous&gt; (plugin:digitalgarden:12669:23)
    at Generator.next (&lt;anonymous&gt;)
    at fulfilled (plugin:digitalgarden:92:24)</pre>[\s\S]*?```/g, "");

    return text
        .split(/\r?\n\s*\r?\n/)
        .map(paragraph => paragraph.trim())
        .filter(paragraph => {

            if (!paragraph) return false;

            // 标题、引用、表格、图片、分隔线不取
            if (/^#{1,6}\s/.test(paragraph)) return false;
            if (/^>/.test(paragraph)) return false;
            if (/^\|/.test(paragraph)) return false;
            if (/^!\[/.test(paragraph)) return false;
            if (/^(-{3,}|\*{3,})$/.test(paragraph)) return false;

            return true;
        })
        .slice(0, limit)
        .map(stripMarkdown)
        .filter(Boolean);
}

// 读取正文（异步）：已有 summary 的文章不必读文件
const articleCards = await Promise.all(
    matchedArticles.map(async page => {

        let excerpt = [];

        const hasSummary = page.summary !== null &&
            page.summary !== undefined &&
            String(page.summary).trim() !== "";

        if (!hasSummary) {

            const file = app.vault.getAbstractFileByPath(page.file.path);

            if (file) {
                try {
                    const content = await app.vault.cachedRead(file);
                    excerpt = extractExcerpt(content, 3);
                } catch (e) {
                    excerpt = [];
                }
            }
        }

        return { page, excerpt };
    })
);

// ==========================
// 7. 渲染
// ==========================

if (articleCards.length > 0) {

    const gallery = createElement(
        dv.container,
        "div",
        "person-article-gallery"
    );

    articleCards.forEach(({ page, excerpt }) => {

        const card = createElement(
            gallery,
            "div",
            "person-article-card"
        );

        // --------------------------
        // 第一、二行：标题、年份、作者
        // --------------------------

        const left = createElement(
            card,
            "div",
            "person-article-card__left"
        );

        const titleText = String(page.title || page.file.name);

        const heading = createElement(
            left,
            "h2",
            "person-article-card__title"
        );

        const titleLink = createElement(
            heading,
            "a",
            "",
            titleText
        );

        titleLink.href = "#";
        titleLink.title = titleText;

        titleLink.addEventListener("click", event => {
            event.preventDefault();
            event.stopPropagation();

            app.workspace.openLinkText(
                page.file.path,
                currentPerson.file.path,
                false
            );
        });

        // 出版年
        if (
            page.datePublished !== null &&
            page.datePublished !== undefined &&
            String(page.datePublished).trim() !== ""
        ) {
            createElement(
                left,
                "div",
                "person-article-card__year",
                String(page.datePublished)
            );
        }

        // 作者
        const authorText = getText(page.author, " / ");

        if (authorText) {
            createElement(
                left,
                "div",
                "person-article-card__author",
                authorText
            );
        }

        // --------------------------
        // 第三行：正文
        // summary 优先，缺失时回退到正文前三段
        // --------------------------

        const summaryText = page.summary
            ? String(page.summary).trim()
            : "";

        let bodyParagraphs = [];

        if (summaryText) {

            bodyParagraphs = summaryText
                .split(/\n+/)
                .map(paragraph => stripMarkdown(paragraph.trim()))
                .filter(Boolean);

        } else {

            bodyParagraphs = excerpt;
        }

        if (bodyParagraphs.length > 0) {

            const right = createElement(
                card,
                "div",
                "person-article-card__right"
            );

            const scrollArea = createElement(
                right,
                "div",
                "person-article-card__excerpt-scroll"
            );

            bodyParagraphs.forEach(paragraph => {
                createElement(
                    scrollArea,
                    "p",
                    "person-article-card__text",
                    paragraph
                );
            });
        }

        // --------------------------
        // 第四行：标签
        // --------------------------

        const tags = getList(page.tags)
            .map(cleanTag)
            .filter(Boolean);

        if (tags.length > 0) {

            const tagWrap = createElement(
                card,
                "div",
                "person-article-card__tags"
            );

            tags.forEach(tag => {

                const matches = pageIndex[normalizeName(tag)] || [];

                // 唯一匹配：可点击的内部链接
                if (matches.length === 1) {

                    const matchedPage = matches[0];

                    const link = createElement(
                        tagWrap,
                        "a",
                        "person-article-card__tag person-article-card__tag--linked",
                        tag
                    );

                    link.href = "#";
                    link.title = "打开页面：" + tag;

                    link.addEventListener("click", event => {
                        event.preventDefault();
                        event.stopPropagation();

                        app.workspace.openLinkText(
                            matchedPage.file.path,
                            currentPerson.file.path,
                            false
                        );
                    });

                }

                // 无匹配或多重匹配：半透明
                else {

                    const label = createElement(
                        tagWrap,
                        "span",
                        "person-article-card__tag person-article-card__tag--missing",
                        tag
                    );

                    label.title = matches.length > 1
                        ? "存在多个同名页面，无法确定链接"
                        : "尚无对应的页面";
                }
            });
        }
    });
}
```