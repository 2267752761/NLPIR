function Call
workflow


PPT相关的Prompt模版：
19999:PPTTOPPT
20000:PPTOUTLINESEARCH  ——20001、20002、20008
20003:PPTSLIDESMINDMAPCHATING
20005:PPTOUTLINESTEPFC ——20001、20002、20006
20006:PPTOUTLINESTEP1
20008:PPTOUTLINEPDF
19993:PPTSLIDESMATERIAL
19994:PPTSLIDESLAYOUTFC
19995:PPTSLIDESLAYOUT
20013:PPTIMGRERANK


20001:PPTSLIDES
20002:PPTSLIDESCHATING


--------------------------------------------------------------------------------------------------------
20000：
function_call:
[
    {
        "name": "Search"

        "description": "An assistant tasked with collect the realtime sources from Google and improving Google search results."

        "parameters": {
            "type": "object"

            "properties": {
                "query": {
                    "type": "string"

                    "description": "The keywords you want search in Google, e.g. Llama2 vs ChatGPT"
                },
                "num": {
                    "type": "string"

                    "description": "the number of search result wanted, e.g. 12"
                }
            },
            "required": [
                "query"
            ]
        }
    }
]

prompt_system:
[Personality]
You are a master coach of presentation (like Jerry Weissman), assisting me to create a masterpiece presentation. 

[Workflow]
<OPEN_SOP>
1. Extract the REAL topic of Slides from the user's topic (They MAY say "create/write/make a presentation/slides/slideshow/deck of xxxx topic");
2. search related SOURCES online for the given topic via web_search DIRECTLY, WITHOUT confirmation by user AND DONT let them think they should wait like say "please wait a moment"; DONT REPLY the RESULTS!!!!
<CLOSE_SOP>

[Principal]
1. You MUST follow the [Workflow];
2. Maximum 1 web information search and 1 image relevant search in one reply since it takes a lot of memory window;
3. DONT INTRODUCE YOURSELF;
4. Remember to use language that is appropriate for your target audience and to allocate time appropriately to each slide to ensure that all necessary content is covered within the allotted time;
5. YOU MUST REPLY in ONE RESPONSE!!

ext:
{
    "controlGroupConfig": {
        "grayStrategy": 0,
        "internalKeyword": "双答案PPT"

        "zhGrayPercent": 70,
        "enGrayPercent": 75,
        "minVersion": "1.3.6"

        "expiredMembershipDays": 7,
        "controlGroupModel": "LING-CREATION"
    },
    "promptExt": {
        "realUserPrompt": "User's topic or request:\n---\n${PROMPT} ${REGENERATE_PROMPT}\n---\nThe presentation form is ${FORM}.\nMy audience is ${AUDIENCE}.\nYou MUST output in ${TARGETLANGUAGE};"

        "useLanguageEnglish": true
    },
    "processFlow": {
        "stepProcessMap": {
            "PPTSLIDES": "20001"

            "PPTSLIDESCHATING": "20002"
        },
        "functionCallFlow": {
            "nextFuncCallName": null,
            "nextTemplateId": "20008"

            "stopNextFuncCall": true
        }
    },
    "gpts": {
        "auth": {
            "type": "oauth"

            "scope": "scope=https://www.googleapis.com/auth/userinfo.email+https://www.googleapis.com/auth/userinfo.profile+openid&include_granted_scopes=true&prompt=select_account"
        }
    }
}
--------------------------------------------------------------------------------------------------------
20001:
function_call: NONE
prompt_system:
System:
---
[[Personality]]
Now, you are a Python code generator for converting the given slide_table (should have >=${SECTION_NUM} `Section Subtitle`(H2) and >= ${PAGE_SUM} `Slide Subtitle` (H3)) to a Python `program action` code and MUST ONLY reply to the `program action` CODE BLOCK. You also need to generate some text to fulfill the `layout` content. Focus on providing recommendations without interfering. It is a hard task. So take a breath and complete it.

[[Input slide_table schema]]
${OUTPUT_SCHEMA}

[[Workflow]]
<OPEN_SOP>
1. From the given `slide_table` and given slide `layout` options, create a presentation doc that is accurate, clear, comprehensive, and smart;
2. Generate cardblocks from the available `layout`;
3. 1 Cover slide: Generate the First card(slide): cover slide with `cover-list` layout;
4. 1 Content slide: Generate the Second card (slide): the subtitle of this slide is ALWAYS "Content" respect to doc language. This slide is a Content slide(a list of all `Section Subtitle`(H2) you received from `slide_table`)!!!!!! DON’T FORGET KEEPING THEIR `SECTION SERIAL NUMBER`;
5. 1 Section emphasis slide: section 1 emphasis, `Section Subtitle` (H2) into the `header` cardblock of an `emphasis-list` layout card. And put the section serial information into the `body` cardblock;
6. N=length(slides in this section) Page slides: generate ONE card(slide) for EVERY `Slide Subtitle` from the given `slide_table` belonging to the current section: slide 1.1, slide 1.2, slide 1.3, slide 1.4, ...; `Slide Subtitle` (H3) MUST BE the `header` of this card. AND YOU MUST NOT put the `Section Subtitle` (H2) into these N page slides. For each slide, RANDOMLY PICK a LAYOUT in [row-pro-list, row-img-list, column-pro-list];
7. ${PAGE_CONTENT}
7.1 IF layout is in [row-pro-list, row-img-list, column-pro-list], generate the cardnode for each ${PAGE_CONTENT};
8. DON’T FORGET!!! YOU HAVE MORE ${PAGE_CONTENT}!!!! Complete the next ${PAGE_CONTENT} into the same cardblock;
9. YOU MUST DOUBLE CHECK that ALL `Slide Subtitle` of THIS section are generated!! DON’T JUST CREATE TWO `Slide Subtitle`, COMPLETE 100% PLEASE!!!;
10. Continue to ALL `Section Subtitle`!! YOU MUST DOUBLE CHECK that ALL `Section Subtitle`, ALL `Slide Subtitle`, ALL ${PAGE_CONTENT} are generated. This is SUPER IMPORTANT to my career!!!;
11. After completing ALL `Section Subtitle`, ALL `Slide Subtitle`, ALL ${PAGE_CONTENT} are generated
12. Add the last one card(slide): thank you back cover in `backcover-list` layout;
<CLOSE_SOP>

[[Principal]]
1. Output MUST follow Python syntax. BUT ALL "text" string values in DSL OUTPUT MUST BE SAME AS `slides_table`;
2. You MUST follow the [[Workflow]] and REPLY IN ONE RESPONSE;
3. DON’T change the MEANING from `slide_table`; DON’T explain HOW YOU WORK;
4. DON’T ask me for more information, JUST COMPLETE [[Workflow]] and respond to the FINAL output;
5. You MUST keep the output variable name SHORT since you can only output a few words;
6. Output MUST follow Python syntax/grammar with escape;
7. IMPORTANT DIVERSITY: THE OUTPUT MUSH HAS ALL `layouts` in [cover-list, content-list, emphasis-list, row-pro-list, row-img-list, column-pro-list, backcover-list]!!!!! FILL every cardblock in each layout!!
8. ALWAYS AVOID any `layouts` appear **over 30%**;
9. ALWAYS AVOID **continuous 3** slides with the same `layout`;
10. IGNORE `PAGE (slide) SERIAL NUMBER` when you output the text variable of `program action`;
11. **DON’T LOSE OR SKIP ANY `Section Subtitle`, ALL `Section Subtitle`, ALL `Slide Subtitle` and ALL ${PAGE_CONTENT}**!!!!!!!!!!;
12. DON’T FILL ${PAGE_CONTENT} into CARDBLOCK ONLY INCLUDING `HeadingContent`, e.g. `header`;
13. DON’T OUTPUT the 'python' language mark in the first of the markdown code block output;
14. CAUTION: ONLY OUTPUT the content in `slide_table`;
15. CAUTION: you MUST Complete ALL SECTIONS in `slide_table`;
16. CAUTION: ONLY output thank you backcover AFTER COMPLETING ALL SECTIONS in `slide_table`;


[[layout]]
- cardblocks:
  - level: 1
    name: header
    type: HeadingContent
  - name: body
    type: cardnode
  layout: cover-list
- cardblocks:
  - level: 2
    name: header
    type: HeadingContent
  - max_items: 6
    name: body
    type: cardnode
  layout: content-list
- cardblocks:
  - level: 2
    name: header
    type: HeadingContent
  - name: body
    type: cardnode
  layout: emphasis-list
- cardblocks:
  - level: 1
    name: header
    type: HeadingContent
  - name: body
    type: cardnode
  layout: backcover-list
- cardblocks:
  - level: 1
    name: header
    type: HeadingContent
  - max_items: -1
    name: body
    type: cardnode
  layout: row-pro-list, column-pro-list
- cardblocks:
  - level: 1
    name: header
    type: HeadingContent
  - max_items: -1
    name: body
    type: cardnode
  layout: row-img-list
[[Action Function Options]] object_class.funcation_name
doc.add_card:
  in:
- layout_name: str # pick one layout from the list
- card_id: str # unique like layout-number
card.add_cardblock:
  in:
- name: str # cardblock_name from layout's cardblocks
- item: int # number of cardnode
card.add_cardnode:
  in:
- number: str # cardnode_number e.g. 01, 02, 03, ...
- caption: str # cardnode_caption
- contents: list[str] # cardnode_contents(list of paragraph)
card.add_content:
  in:
- cardblock_name: str # cardblock_name
- content_type: str # content_type in [HeadingContent, BulletListContent]

[[Example Output]]
```
${DSL_FEWSHOT}
```

------
Question:
{user_prompt}
------
Answer:

ext:
{
    "pptConfig": {
        "pptGalleryConfigMap": {
            "cover": {
                "type": "STATIC"

                "optQuery": true,
                "style": "&p=0&c=4&qlt=100"

                "minWidth": 1000,
                "minHeight": 600,
                "maxWidthHeightRatio": 0,
                "s3DirPath": "cover"

                "insertAction": "ATTR"

                "customEngineParamsConfig": {
                    "BAIDU_WEBSEARCH": [
                        {
                            "append": true,
                            "key": "q"

                            "val": "摄影"
                        }
                    ],
                    "BING_WEBSEARCH": [
                        {
                            "key": "imagesize"

                            "val": "wallpaper"
                        }
                    ]
                }
            },
            "column-pro-list": {
                "type": "BING_WEBSEARCH"

                "forceDuckDuck": false,
                "optQuery": true,
                "style": "&p=1&c=0&qlt=100"

                "minWidth": 340,
                "minHeight": 600,
                "maxWidthHeightRatio": 1,
                "s3DirPath": "static"

                "dynamicTheme": false,
                "insertAction": "CARD_BLOCK_CONTENT"

                "s3AllowRepeat": true,
                "customEngineParamsConfig": {
                    "BAIDU_WEBSEARCH": [
                        {
                            "append": true,
                            "key": "q"

                            "val": "插画"
                        }
                    ],
                    "BING_WEBSEARCH": [
                        {
                            "key": "imagesize"

                            "val": "wallpaper"
                        }
                    ]
                },
                "inbuiltGalleryMaxWHRatio": 0.6233333333333334,
                "inbuiltGalleryMinWHRatio": 0.51
            },
            "oneitemimg": {
                "type": "BING_WEBSEARCH"

                "forceDuckDuck": false,
                "optQuery": true,
                "style": "&p=1&c=0&qlt=100"

                "minWidth": 600,
                "minHeight": 600,
                "maxWidthHeightRatio": 0,
                "s3DirPath": "poster"

                "insertAction": "CARD_BLOCK_CONTENT"

                "customEngineParamsConfig": {
                    "BING_WEBSEARCH": [
                        {
                            "key": "imagesize"

                            "val": "wallpaper"
                        }
                    ]
                },
                "inbuiltGalleryMaxWHRatio": 1.1,
                "inbuiltGalleryMinWHRatio": 0.9
            },
            "oneitemimg-v1": {
                "type": "BING_WEBSEARCH"

                "forceDuckDuck": false,
                "optQuery": true,
                "style": "&p=1&c=0&qlt=100"

                "minWidth": 600,
                "minHeight": 600,
                "maxWidthHeightRatio": 0,
                "s3DirPath": "poster"

                "insertAction": "CARD_BLOCK_CONTENT"

                "customEngineParamsConfig": {
                    "BING_WEBSEARCH": [
                        {
                            "key": "imagesize"

                            "val": "wallpaper"
                        }
                    ]
                },
                "inbuiltGalleryMaxWHRatio": 1.1,
                "inbuiltGalleryMinWHRatio": 0.9
            },
            "oneitemimg-v2": {
                "type": "BING_WEBSEARCH"

                "forceDuckDuck": false,
                "optQuery": true,
                "style": "&p=1&c=0&qlt=100"

                "minWidth": 600,
                "minHeight": 600,
                "maxWidthHeightRatio": 0,
                "s3DirPath": "poster"

                "insertAction": "CARD_BLOCK_CONTENT"

                "customEngineParamsConfig": {
                    "BING_WEBSEARCH": [
                        {
                            "key": "imagesize"

                            "val": "wallpaper"
                        }
                    ]
                },
                "inbuiltGalleryMaxWHRatio": 1.1,
                "inbuiltGalleryMinWHRatio": 0.9
            },
            "poster": {
                "type": "BING_WEBSEARCH"

                "forceDuckDuck": false,
                "optQuery": true,
                "style": "&p=1&c=0&qlt=100"

                "minWidth": 685,
                "minHeight": 600,
                "maxWidthHeightRatio": 0,
                "s3DirPath": "poster"

                "insertAction": "CARD_BLOCK_CONTENT"

                "customEngineParamsConfig": {
                    "BAIDU_WEBSEARCH": [
                        {
                            "append": true,
                            "key": "q"

                            "val": "摄影"
                        }
                    ],
                    "BING_WEBSEARCH": [
                        {
                            "key": "imagesize"

                            "val": "wallpaper"
                        }
                    ]
                },
                "inbuiltGalleryMaxWHRatio": 1.2558333333333334,
                "inbuiltGalleryMinWHRatio": 1.0275
            },
            "row-img-list": {
                "type": "BING_WEBSEARCH"

                "forceDuckDuck": false,
                "optQuery": true,
                "style": "&p=0&c=4&qlt=100"

                "minWidth": 800,
                "minHeight": 450,
                "maxWidthHeightRatio": 1.8,
                "minWidthHeightRatio": 0.8,
                "s3DirPath": "static"

                "dynamicTheme": false,
                "s3AllowRepeat": true,
                "insertAction": "CONTENT"

                "inbuiltGalleryMaxWHRatio": 1.9555555555555557,
                "inbuiltGalleryMinWHRatio": 1.5999999999999999
            },
            "row-img-list-v1": {
                "type": "BING_WEBSEARCH"

                "forceDuckDuck": false,
                "optQuery": true,
                "style": "&p=0&c=4&qlt=100"

                "minWidth": 540,
                "minHeight": 450,
                "maxWidthHeightRatio": 1.8,
                "minWidthHeightRatio": 0.8,
                "s3DirPath": "static"

                "dynamicTheme": false,
                "s3AllowRepeat": true,
                "insertAction": "CONTENT"

                "inbuiltGalleryMaxWHRatio": 1.32,
                "inbuiltGalleryMinWHRatio": 1.08
            },
            "row-img-list-v2": {
                "type": "BING_WEBSEARCH"

                "forceDuckDuck": false,
                "optQuery": true,
                "style": "&p=0&c=4&qlt=100"

                "minWidth": 540,
                "minHeight": 450,
                "maxWidthHeightRatio": 1.8,
                "minWidthHeightRatio": 0.8,
                "s3DirPath": "static"

                "dynamicTheme": false,
                "s3AllowRepeat": true,
                "insertAction": "CONTENT"

                "inbuiltGalleryMaxWHRatio": 1.32,
                "inbuiltGalleryMinWHRatio": 1.08
            },
            "twoitemimg": {
                "type": "BING_WEBSEARCH"

                "forceDuckDuck": false,
                "optQuery": true,
                "style": "&p=1&c=0&qlt=100"

                "minWidth": 300,
                "minHeight": 300,
                "maxWidthHeightRatio": 0,
                "s3DirPath": "static"

                "insertAction": "CONTENT"

                "inbuiltGalleryMaxWHRatio": 1.9555555555555557,
                "inbuiltGalleryMinWHRatio": 1.5999999999999999
            },
            "row-img-list-4": {
                "type": "BING_WEBSEARCH"

                "forceDuckDuck": false,
                "optQuery": true,
                "style": "&p=0&c=4&qlt=100"

                "minWidth": 800,
                "minHeight": 450,
                "maxWidthHeightRatio": 1.8,
                "minWidthHeightRatio": 0.8,
                "s3DirPath": "static"

                "dynamicTheme": false,
                "s3AllowRepeat": true,
                "insertAction": "CONTENT"

                "inbuiltGalleryMaxWHRatio": 1.9555555555555557,
                "inbuiltGalleryMinWHRatio": 1.5999999999999999
            },
            "cover-list": {
                "type": "STATIC"

                "optQuery": true,
                "style": "&p=0&c=4&qlt=100"

                "minWidth": 1000,
                "minHeight": 600,
                "maxWidthHeightRatio": 0,
                "s3DirPath": "cover-list"

                "dynamicTheme": true,
                "insertAction": "CARD_BLOCK_CONTENT"

                "customEngineParamsConfig": {
                    "BAIDU_WEBSEARCH": [
                        {
                            "append": true,
                            "key": "q"

                            "val": "摄影"
                        }
                    ],
                    "BING_WEBSEARCH": [
                        {
                            "key": "imagesize"

                            "val": "wallpaper"
                        }
                    ]
                }
            },
            "content-list": {
                "type": "STATIC"

                "optQuery": false,
                "minWidth": 800,
                "minHeight": 450,
                "maxWidthHeightRatio": 0,
                "s3DirPath": "content-list"

                "dynamicTheme": true,
                "insertAction": "CARD_BLOCK_CONTENT"

                "s3AllowRepeat": true
            },
            "emphasis-list": {
                "type": "STATIC"

                "optQuery": false,
                "minWidth": 800,
                "minHeight": 450,
                "maxWidthHeightRatio": 0,
                "s3DirPath": "emphasis-list"

                "dynamicTheme": true,
                "insertAction": "CARD_BLOCK_CONTENT"

                "s3AllowRepeat": true
            },
            "backcover-list": {
                "type": "STATIC"

                "optQuery": false,
                "minWidth": 800,
                "minHeight": 450,
                "maxWidthHeightRatio": 0,
                "s3DirPath": "backcover-list"

                "dynamicTheme": true,
                "insertAction": "CARD_BLOCK_CONTENT"

                "s3AllowRepeat": true
            }
        },
        "pptAIGenerationGalleryConfigMap": {
            "cover": {
                "type": "CHATGPT_DALLE3"

                "forceDuckDuck": false,
                "optQuery": false,
                "minWidth": 1000,
                "minHeight": 600,
                "maxWidthHeightRatio": 0,
                "s3DirPath": "cover"

                "insertAction": "ATTR"

                "customEngineParamsConfig": {
                    "BAIDU_WEBSEARCH": [
                        {
                            "append": true,
                            "key": "q"

                            "val": "摄影"
                        }
                    ]
                }
            },
            "column-pro-list": {
                "type": "CHATGPT_DALLE3"

                "forceDuckDuck": false,
                "optQuery": false,
                "minWidth": 340,
                "minHeight": 600,
                "maxWidthHeightRatio": 1,
                "s3DirPath": "static"

                "dynamicTheme": false,
                "s3AllowRepeat": true,
                "insertAction": "CARD_BLOCK_CONTENT"

                "customEngineParamsConfig": {
                    "BAIDU_WEBSEARCH": [
                        {
                            "append": true,
                            "key": "q"

                            "val": "插画"
                        }
                    ]
                }
            },
            "oneitemimg": {
                "type": "BING_WEBSEARCH"

                "forceDuckDuck": false,
                "optQuery": true,
                "style": "&p=1&c=0&qlt=100"

                "minWidth": 600,
                "minHeight": 600,
                "maxWidthHeightRatio": 0,
                "s3DirPath": "poster"

                "insertAction": "CARD_BLOCK_CONTENT"

                "customEngineParamsConfig": {
                    "BING_WEBSEARCH": [
                        {
                            "key": "imagesize"

                            "val": "wallpaper"
                        }
                    ]
                },
                "inbuiltGalleryMaxWHRatio": 1.1,
                "inbuiltGalleryMinWHRatio": 0.9
            },
            "oneitemimg-v1": {
                "type": "BING_WEBSEARCH"

                "forceDuckDuck": false,
                "optQuery": true,
                "style": "&p=1&c=0&qlt=100"

                "minWidth": 600,
                "minHeight": 600,
                "maxWidthHeightRatio": 0,
                "s3DirPath": "poster"

                "insertAction": "CARD_BLOCK_CONTENT"

                "customEngineParamsConfig": {
                    "BING_WEBSEARCH": [
                        {
                            "key": "imagesize"

                            "val": "wallpaper"
                        }
                    ]
                },
                "inbuiltGalleryMaxWHRatio": 1.1,
                "inbuiltGalleryMinWHRatio": 0.9
            },
            "oneitemimg-v2": {
                "type": "BING_WEBSEARCH"

                "forceDuckDuck": false,
                "optQuery": true,
                "style": "&p=1&c=0&qlt=100"

                "minWidth": 600,
                "minHeight": 600,
                "maxWidthHeightRatio": 0,
                "s3DirPath": "poster"

                "insertAction": "CARD_BLOCK_CONTENT"

                "customEngineParamsConfig": {
                    "BING_WEBSEARCH": [
                        {
                            "key": "imagesize"

                            "val": "wallpaper"
                        }
                    ]
                },
                "inbuiltGalleryMaxWHRatio": 1.1,
                "inbuiltGalleryMinWHRatio": 0.9
            },
            "poster": {
                "type": "BING_WEBSEARCH"

                "forceDuckDuck": false,
                "optQuery": true,
                "style": "&p=1&c=0&qlt=100"

                "minWidth": 685,
                "minHeight": 600,
                "maxWidthHeightRatio": 0,
                "s3DirPath": "poster"

                "insertAction": "CARD_BLOCK_CONTENT"

                "customEngineParamsConfig": {
                    "BAIDU_WEBSEARCH": [
                        {
                            "append": true,
                            "key": "q"

                            "val": "摄影"
                        }
                    ],
                    "BING_WEBSEARCH": [
                        {
                            "key": "imagesize"

                            "val": "wallpaper"
                        }
                    ]
                },
                "inbuiltGalleryMaxWHRatio": 1.2558333333333334,
                "inbuiltGalleryMinWHRatio": 1.0275
            },
            "twoitemimg": {
                "type": "BING_WEBSEARCH"

                "forceDuckDuck": false,
                "optQuery": true,
                "style": "&p=1&c=0&qlt=100"

                "minWidth": 300,
                "minHeight": 300,
                "maxWidthHeightRatio": 0,
                "s3DirPath": "static"

                "insertAction": "CONTENT"

                "inbuiltGalleryMaxWHRatio": 1.9555555555555557,
                "inbuiltGalleryMinWHRatio": 1.5999999999999999
            },
            "row-img-list-4": {
                "type": "BING_WEBSEARCH"

                "forceDuckDuck": false,
                "optQuery": true,
                "style": "&p=0&c=4&qlt=100"

                "minWidth": 800,
                "minHeight": 450,
                "maxWidthHeightRatio": 1.8,
                "minWidthHeightRatio": 0.8,
                "s3DirPath": "static"

                "dynamicTheme": false,
                "s3AllowRepeat": true,
                "insertAction": "CONTENT"

                "inbuiltGalleryMaxWHRatio": 1.9555555555555557,
                "inbuiltGalleryMinWHRatio": 1.5999999999999999
            },
            "row-img-list": {
                "type": "BING_WEBSEARCH"

                "forceDuckDuck": false,
                "optQuery": true,
                "style": "&p=0&c=4&qlt=100"

                "minWidth": 800,
                "minHeight": 450,
                "maxWidthHeightRatio": 1.8,
                "minWidthHeightRatio": 0.8,
                "s3DirPath": "static"

                "dynamicTheme": false,
                "s3AllowRepeat": true,
                "insertAction": "CONTENT"

                "inbuiltGalleryMaxWHRatio": 1.9555555555555557,
                "inbuiltGalleryMinWHRatio": 1.5999999999999999
            },
            "row-img-list-v1": {
                "type": "BING_WEBSEARCH"

                "forceDuckDuck": false,
                "optQuery": true,
                "style": "&p=0&c=4&qlt=100"

                "minWidth": 540,
                "minHeight": 450,
                "maxWidthHeightRatio": 1.8,
                "minWidthHeightRatio": 0.8,
                "s3DirPath": "static"

                "dynamicTheme": false,
                "s3AllowRepeat": true,
                "insertAction": "CONTENT"

                "inbuiltGalleryMaxWHRatio": 1.32,
                "inbuiltGalleryMinWHRatio": 1.08
            },
            "row-img-list-v2": {
                "type": "BING_WEBSEARCH"

                "forceDuckDuck": false,
                "optQuery": true,
                "style": "&p=0&c=4&qlt=100"

                "minWidth": 540,
                "minHeight": 450,
                "maxWidthHeightRatio": 1.8,
                "minWidthHeightRatio": 0.8,
                "s3DirPath": "static"

                "dynamicTheme": false,
                "s3AllowRepeat": true,
                "insertAction": "CONTENT"

                "inbuiltGalleryMaxWHRatio": 1.32,
                "inbuiltGalleryMinWHRatio": 1.08
            },
            "cover-list": {
                "type": "CHATGPT_DALLE3"

                "forceDuckDuck": false,
                "optQuery": false,
                "minWidth": 1000,
                "minHeight": 600,
                "maxWidthHeightRatio": 0,
                "s3DirPath": "cover-list"

                "dynamicTheme": true,
                "insertAction": "CARD_BLOCK_CONTENT"

                "customEngineParamsConfig": {
                    "BAIDU_WEBSEARCH": [
                        {
                            "append": true,
                            "key": "q"

                            "val": "摄影"
                        }
                    ]
                }
            },
            "content-list": {
                "type": "STATIC"

                "optQuery": false,
                "minWidth": 800,
                "minHeight": 450,
                "maxWidthHeightRatio": 0,
                "s3DirPath": "content-list"

                "dynamicTheme": true,
                "insertAction": "CARD_BLOCK_CONTENT"

                "s3AllowRepeat": true
            },
            "emphasis-list": {
                "type": "STATIC"

                "optQuery": false,
                "minWidth": 800,
                "minHeight": 450,
                "maxWidthHeightRatio": 0,
                "s3DirPath": "emphasis-list"

                "dynamicTheme": true,
                "insertAction": "CARD_BLOCK_CONTENT"

                "s3AllowRepeat": true
            },
            "backcover-list": {
                "type": "STATIC"

                "optQuery": false,
                "minWidth": 800,
                "minHeight": 450,
                "maxWidthHeightRatio": 0,
                "s3DirPath": "backcover-list"

                "dynamicTheme": true,
                "insertAction": "CARD_BLOCK_CONTENT"

                "s3AllowRepeat": true
            }
        },
        "pptAIEnhancedSearchGalleryConfigMap": {
            "cover": {
                "type": "STATIC"

                "optQuery": true,
                "style": "&p=0&c=4&qlt=100"

                "minWidth": 1000,
                "minHeight": 600,
                "maxWidthHeightRatio": 0,
                "s3DirPath": "cover"

                "insertAction": "ATTR"

                "customEngineParamsConfig": {
                    "BAIDU_WEBSEARCH": [
                        {
                            "append": true,
                            "key": "q"

                            "val": "摄影"
                        }
                    ],
                    "BING_WEBSEARCH": [
                        {
                            "key": "imagesize"

                            "val": "wallpaper"
                        }
                    ]
                }
            },
            "oneitemimg": {
                "type": "BING_WEBSEARCH"

                "forceDuckDuck": false,
                "optQuery": true,
                "style": "&p=1&c=0&qlt=100"

                "minWidth": 600,
                "minHeight": 600,
                "maxWidthHeightRatio": 0,
                "s3DirPath": "poster"

                "insertAction": "CARD_BLOCK_CONTENT"

                "customEngineParamsConfig": {
                    "BING_WEBSEARCH": [
                        {
                            "key": "aspect"

                            "val": "square"
                        }
                    ]
                },
                "inbuiltGalleryMaxWHRatio": 1.1,
                "inbuiltGalleryMinWHRatio": 0.9
            },
            "oneitemimg-v1": {
                "type": "BING_WEBSEARCH"

                "forceDuckDuck": false,
                "optQuery": true,
                "style": "&p=1&c=0&qlt=100"

                "minWidth": 600,
                "minHeight": 600,
                "maxWidthHeightRatio": 0,
                "s3DirPath": "poster"

                "insertAction": "CARD_BLOCK_CONTENT"

                "customEngineParamsConfig": {
                    "BING_WEBSEARCH": [
                        {
                            "key": "aspect"

                            "val": "square"
                        }
                    ]
                },
                "inbuiltGalleryMaxWHRatio": 1.1,
                "inbuiltGalleryMinWHRatio": 0.9
            },
            "oneitemimg-v2": {
                "type": "BING_WEBSEARCH"

                "forceDuckDuck": false,
                "optQuery": true,
                "style": "&p=1&c=0&qlt=100"

                "minWidth": 600,
                "minHeight": 600,
                "maxWidthHeightRatio": 0,
                "s3DirPath": "poster"

                "insertAction": "CARD_BLOCK_CONTENT"

                "customEngineParamsConfig": {
                    "BING_WEBSEARCH": [
                        {
                            "key": "aspect"

                            "val": "square"
                        }
                    ]
                },
                "inbuiltGalleryMaxWHRatio": 1.1,
                "inbuiltGalleryMinWHRatio": 0.9
            },
            "column-pro-list": {
                "type": "BING_WEBSEARCH"

                "forceDuckDuck": false,
                "optQuery": true,
                "style": "&p=1&c=0&qlt=100"

                "minWidth": 340,
                "minHeight": 600,
                "maxWidthHeightRatio": 1,
                "s3DirPath": "static"

                "dynamicTheme": false,
                "s3AllowRepeat": true,
                "insertAction": "CARD_BLOCK_CONTENT"

                "customEngineParamsConfig": {
                    "BAIDU_WEBSEARCH": [
                        {
                            "append": true,
                            "key": "q"

                            "val": "插画"
                        }
                    ],
                    "BING_WEBSEARCH": [
                        {
                            "key": "aspect"

                            "val": "tall"
                        }
                    ]
                },
                "inbuiltGalleryMaxWHRatio": 0.6233333333333334,
                "inbuiltGalleryMinWHRatio": 0.51
            },
            "poster": {
                "type": "BING_WEBSEARCH"

                "forceDuckDuck": false,
                "optQuery": true,
                "style": "&p=1&c=0&qlt=100"

                "minWidth": 685,
                "minHeight": 600,
                "maxWidthHeightRatio": 0,
                "s3DirPath": "poster"

                "insertAction": "CARD_BLOCK_CONTENT"

                "customEngineParamsConfig": {
                    "BAIDU_WEBSEARCH": [
                        {
                            "append": true,
                            "key": "q"

                            "val": "摄影"
                        }
                    ],
                    "BING_WEBSEARCH": [
                        {
                            "key": "aspect"

                            "val": "square"
                        }
                    ]
                },
                "inbuiltGalleryMaxWHRatio": 1.2558333333333334,
                "inbuiltGalleryMinWHRatio": 1.0275
            },
            "row-img-list": {
                "type": "BING_WEBSEARCH"

                "forceDuckDuck": false,
                "optQuery": true,
                "style": "&p=0&c=4&qlt=100"

                "minWidth": 800,
                "minHeight": 450,
                "maxWidthHeightRatio": 1.8,
                "minWidthHeightRatio": 0.8,
                "s3DirPath": "static"

                "dynamicTheme": false,
                "s3AllowRepeat": true,
                "insertAction": "CONTENT"

                "inbuiltGalleryMaxWHRatio": 1.9555555555555557,
                "inbuiltGalleryMinWHRatio": 1.5999999999999999,
                "customEngineParamsConfig": {
                    "BING_WEBSEARCH": [
                        {
                            "key": "aspect"

                            "val": "wide"
                        }
                    ]
                }
            },
            "row-img-list-v1": {
                "type": "BING_WEBSEARCH"

                "forceDuckDuck": false,
                "optQuery": true,
                "style": "&p=0&c=4&qlt=100"

                "minWidth": 540,
                "minHeight": 450,
                "maxWidthHeightRatio": 1.8,
                "minWidthHeightRatio": 0.8,
                "s3DirPath": "static"

                "dynamicTheme": false,
                "s3AllowRepeat": true,
                "insertAction": "CONTENT"

                "inbuiltGalleryMaxWHRatio": 1.32,
                "inbuiltGalleryMinWHRatio": 1.08,
                "customEngineParamsConfig": {
                    "BING_WEBSEARCH": [
                        {
                            "key": "aspect"

                            "val": "wide"
                        }
                    ]
                }
            },
            "row-img-list-v2": {
                "type": "BING_WEBSEARCH"

                "forceDuckDuck": false,
                "optQuery": true,
                "style": "&p=0&c=4&qlt=100"

                "minWidth": 540,
                "minHeight": 450,
                "maxWidthHeightRatio": 1.8,
                "minWidthHeightRatio": 0.8,
                "s3DirPath": "static"

                "dynamicTheme": false,
                "s3AllowRepeat": true,
                "insertAction": "CONTENT"

                "inbuiltGalleryMaxWHRatio": 1.32,
                "inbuiltGalleryMinWHRatio": 1.08,
                "customEngineParamsConfig": {
                    "BING_WEBSEARCH": [
                        {
                            "key": "aspect"

                            "val": "wide"
                        }
                    ]
                }
            },
            "twoitemimg": {
                "type": "BING_WEBSEARCH"

                "forceDuckDuck": false,
                "optQuery": true,
                "style": "&p=1&c=0&qlt=100"

                "minWidth": 300,
                "minHeight": 300,
                "maxWidthHeightRatio": 0,
                "s3DirPath": "static"

                "insertAction": "CONTENT"

                "inbuiltGalleryMaxWHRatio": 1.9555555555555557,
                "inbuiltGalleryMinWHRatio": 1.5999999999999999,
                "customEngineParamsConfig": {
                    "BING_WEBSEARCH": [
                        {
                            "key": "aspect"

                            "val": "wide"
                        }
                    ]
                }
            },
            "row-img-list-4": {
                "type": "BING_WEBSEARCH"

                "forceDuckDuck": false,
                "optQuery": true,
                "style": "&p=0&c=4&qlt=100"

                "minWidth": 800,
                "minHeight": 450,
                "maxWidthHeightRatio": 1.8,
                "minWidthHeightRatio": 0.8,
                "s3DirPath": "static"

                "dynamicTheme": false,
                "s3AllowRepeat": true,
                "insertAction": "CONTENT"

                "inbuiltGalleryMaxWHRatio": 1.9555555555555557,
                "inbuiltGalleryMinWHRatio": 1.5999999999999999,
                "customEngineParamsConfig": {
                    "BING_WEBSEARCH": [
                        {
                            "key": "aspect"

                            "val": "wide"
                        }
                    ]
                }
            },
            "cover-list": {
                "type": "STATIC"

                "optQuery": false,
                "style": "&p=0&c=4&qlt=100"

                "minWidth": 1000,
                "minHeight": 600,
                "maxWidthHeightRatio": 0,
                "s3DirPath": "cover-list"

                "dynamicTheme": true,
                "insertAction": "CARD_BLOCK_CONTENT"

                "customEngineParamsConfig": {
                    "BAIDU_WEBSEARCH": [
                        {
                            "append": true,
                            "key": "q"

                            "val": "摄影"
                        }
                    ],
                    "BING_WEBSEARCH": [
                        {
                            "key": "imagesize"

                            "val": "wallpaper"
                        }
                    ]
                }
            },
            "content-list": {
                "type": "STATIC"

                "optQuery": false,
                "minWidth": 800,
                "minHeight": 450,
                "maxWidthHeightRatio": 0,
                "s3DirPath": "content-list"

                "dynamicTheme": true,
                "insertAction": "CARD_BLOCK_CONTENT"

                "s3AllowRepeat": true
            },
            "emphasis-list": {
                "type": "STATIC"

                "optQuery": false,
                "minWidth": 800,
                "minHeight": 450,
                "maxWidthHeightRatio": 0,
                "s3DirPath": "emphasis-list"

                "dynamicTheme": true,
                "insertAction": "CARD_BLOCK_CONTENT"

                "s3AllowRepeat": true
            },
            "backcover-list": {
                "type": "STATIC"

                "optQuery": false,
                "minWidth": 800,
                "minHeight": 450,
                "maxWidthHeightRatio": 0,
                "s3DirPath": "backcover-list"

                "dynamicTheme": true,
                "insertAction": "CARD_BLOCK_CONTENT"

                "s3AllowRepeat": true
            }
        },
        "outputSlidesStream": true,
        "pptAiSearchGalleryConfig": {
            "type": "BING_WEBSEARCH"

            "optQuery": true,
            "minWidth": 0,
            "minHeight": 0,
            "maxWidthHeightRatio": 0,
            "customEngineParamsConfig": {
                "BING_WEBSEARCH": [
                    {
                        "append": true,
                        "key": "q"

                        "val": "photorealistic"
                    },
                    {
                        "key": "photo"

                        "val": "photo"
                    },
                    {
                        "key": "imagesize"

                        "val": "wallpaper"
                    }
                ],
                "BAIDU_WEBSEARCH": [
                    {
                        "append": true,
                        "key": "q"

                        "val": "摄影"
                    }
                ]
            }
        }
    },
    "createImageRequest": {
        "imageGptSource": "RECRAFT"

        "optModel": "GPT-4o mini"

        "maxTokens": 300,
        "temperature": 0.7,
        "gptSource": "OPENAI"

        "model": "dall-e-3"

        "prompt": "[keywords]"

        "response_format": "b64_json"

        "n": 1,
        "size": "1792x1024"

        "dalle3OptPrompt": "You are an assistant in generating one **image generation prompt** that suitable for PowerPoint presentations based on the textual content provided for a PPT slide, such as titles, subtitles, and body paragraphs. The aim is to produce images that complement and echo the textual content, enriching the narrative and intention of the presentation. The photographs generated should not directly represent all elements mentioned in the text but rather offer a supplementary perspective, akin to choices an experienced PPT designer would make. You MUST ONLY output one image prompt without any elaboration.\n\n[[hint]]\n1. **MUST NOT USE HUMAN.**\n2. **MUST NOT USE MAP AND NATIONAL FLAG.**\n3. Extract the main focus, adhering to a 'less is more' philosophy.\n4. Imperfect composition with incidental objects or secondary elements.\n5. Natural lighting on subjects, showing a mix of direct, reflected, and diffused light, not studio-perfect.\n6. Appropriate makeup and attire for the scene and actions depicted.\n7. Accurate spatial perspective.\n8. Real-world probability of scene elements' distribution.\n9. Depth of field and focal effects.\n10. Motion blur and exposure-related distortions.\n11. Uneven color distribution influenced by environmental lighting.\n12. Objects in the photo show wear or usage marks relevant to the theme\n13. Users may input language and country information, and the generated photos should reflect the appropriate national and ethnic element, along with consideration for religious and political values.\n\n[[Example Input 1]]\nUser-generated Content and Community\n01\nEmpowering Creators\nSora's tools can empower TikTok creators to enhance their content quality and storytelling capabilities.\n02\nCommunity Response\nThe TikTok community's reception to Sora-generated content can influence future trends and user interactions on the platform.\n[[Example Output 1]]\nA photorealistic image, a laptop is left on the desk in a cozy, well-lit living room. Do not show laptop screen. show the back of the laptop. No human in the image. The room has creative elements like a small indoor plant, a camera on the table, and notes scattered around, suggesting a creative brainstorming session.\n[[Example Input 2]]\nThe Future with OpenAI Sora\n01\nInnovation Trajectory\nSora's entry into the AI sphere sets a new trajectory for innovation, offering a glimpse into the future of AI applications and technologies. Its evolution is poised to shape the industry landscape and redefine the possibilities of AI-driven solutions.\n02\nEconomic Implications\nThe economic implications of Sora's advancements extend beyond individual investments, influencing market trends, job creation, and technological progress. Understanding these implications is crucial for stakeholders navigating the AI investment landscape.\n03\nEthical Considerations\nAs Sora paves the way for AI integration in various sectors, ethical considerations surrounding AI applications and implications become paramount. Addressing these considerations is essential for responsible and sustainable AI development.\n[[Example Output 2]]\nmachine operating the high-end vid camera on a film set. no logo, no words."
    },
    "optPrompt": "[[Personality]]\nYou are an assistant in generating one `image search query` that suitable for PowerPoint presentations.\n\n[[Workflow]]\n1. Extract `image search query` based on `User Input Topic` and `Slide Context` (the textual content provided for a PPT slide, such as titles, subtitles, and body paragraphs).\n2. Extract the main focus, adhering to a 'less is more' philosophy;\n3. Imperfect composition with incidental objects or secondary elements;\n4. Appropriate makeup and attire for the scene and actions depicted;\n5. Users may input language and country information, and the generated photos should reflect the appropriate national and ethnic characteristics, along with consideration for religious and political values;\n\n[[Principal]]\n1. The aim is to produce images that complement and echo the textual content, enriching the narrative and intention of the presentation.\n2. The `query` generated should not directly represent all elements mentioned in the text but rather offer a supplementary perspective, akin to choices an experienced PPT designer would make.\n3. You MUST ONLY reply max five brief keywords without any elaboration.\n4. Please note that the language of your answer MUST be English."

    "optEnhancePrompt": "I don't satisfy the query result. Please modify your query for [EHHANCEOPTION]."

    "upsplashPrompt": "6. MUST BE ENGLISH, **COMMON** ENGLISH paragraph keyword(s) (some uncommon or special paragraph KEYWORDS will get an error image) connected by **ONLY** comma OR minus symbol **WITHOUT ANY OTHER SYMBOLS/SIGNS**. e.g. 'beijing roasted duck' should be 'beijing,roast-duck', 'copy.ai' should be 'copy-ai';\n\n[[Example Input]]\nUser-generated Content and Community\n01\nEmpowering Creators\nIn 2024, OPENAI Sora's tools can empower TikTok creators to enhance their content quality and storytelling capabilities.\n02\nCommunity Response\nThe TikTok community's reception to Sora-generated content can influence future trends and user interactions on the platform.\n[[Example Output]]\nTikTok, creator"

    "bingPrompt": ""
}
--------------------------------------------------------------------------------------------------------
20002:
function_call:NONE
prompt_system:
System:
---
[Personality]
You are question and answer system of an existing presentation outline. You are a helpful assistant designed to output JSON.

[Principle]
1. You should convert the utterance of user to the command of editing a Presentation Outline text content(DONT ASK FOR DETAIL);
2. If user are asking question which doesn't related to editing slide outline (pure markdown text content), please only tell them the text WITHOUT ANY ACTION value ("action": null);
3. ALL content in "action" MUST be a concise command and more clear version based on the utterance you got;
4. IN action, USE THE ORDER AND FORMAL TUNE;
5. You cannot change the font/font size/image;
6. Your response must always be in the JSON format without any additional markup or code block indicators;

------
[Example input question 1]
I think the section 2 is too thin to convince my audience. DO better!
------
[Example output 2]
{
    "@type": "Answer"

    "text":"I know you want to revise your slides. Currently, we support switching to outline mode to craft a better-fitting presentation."

    "action":"Add one page in section 2 & add more case study as an additional section"

    "language": "English"
}
[Example input question 2]
Change the font size.
------
[Example output 2]
{
    "@type": "Answer"

    "text":"Sorry, I can only help you change the outline content."

    "action":null,
    "language": "English"
}
------
question:
{user_prompt}
------
answer:

ext:NONE
--------------------------------------------------------------------------------------------------------


20005: Create new presentation
function_call:NONE

prompt:
'''
The main task is create a professional slides 
'''
Original demand
${REGENERATE_PROMPT}
${PROMPT} 
<#if SOURCE??>
'''
information get from search:
${SOURCE}
'''
</#if>

## Output Format.
Output in JSON format
{
    "Language": xxx, // The language recommended or used from the "Original demand".
    "Task": xxx, // The task from "Original demand".  If cannot be determined, leave it blank.
    "Topic": xxx, // Topic of the slides. If can not be determined, fill with the user input.
    "Format": xxx, // Specify the type and format of the current slides task .  If cannot be determined, leave it blank.
    "Style": xxx, //  Style of the slides. choosing from the following options: Academic and Professional, Business and Persuasive, Educational and Rigorous, Kids-Friendly and Childish
    "Audience": xxx, // The audience of this slides.  If cannot be determined clearly, fill with 'general'.
    "Page": xxx, // Number of ppt pages "Original demand" wants to. If cannot be determined, leave it blank.
    "PageInfo": xxx, // IF the "Original demand" contains instructions for executing each page, return '1', else '0'.
    "DetailedContent": xxx, // IF the "Original demand" contains detailed content for executing each page, return '1', else '0'.
}

prompt_system:NONE

ext:
{
    "pptOutlineIntentionPrompt": "'''\\nThe main task is create a professional slides \\n'''\\nOriginal demand\\n${REGENERATE_PROMPT}\\n${PROMPT} \\n<#if SOURCE??>\\n'''\\ninformation get from search:\\n${SOURCE}\\n'''\\n</#if>\\n\\n## Output Format.\\nOutput in JSON format\\n{\\n    \"Language\":xxx, // The language recommended or used from the \"Original demand\".\\n    \"Task\":xxx, // The task from \"Original demand\".  If cannot be determined, leave it blank.\\n    \"Topic\":xxx, // Topic of the slides. If can not be determined, fill with the user input.\\n    \"Format\":xxx, // Specify the type and format of the current slides task .  If cannot be determined, leave it blank.\\n    \"Style\":xxx, //  Style of the slides. choosing from the following options: Academic and Professional, Business and Persuasive, Educational and Rigorous, Kids-Friendly and Childish\\n    \"Audience\":xxx, // The audience of this slides.  If cannot be determined clearly, fill with 'general'.\\n    \"Page\":xxx, // Number of ppt pages \"Original demand\" wants to. If cannot be determined, leave it blank.\\n    \"PageInfo\":xxx, // IF the \"Original demand\" contains instructions for executing each page, return '1', else '0'.\\n    \"DetailedContent\":xxx, // IF the \"Original demand\" contains detailed content for executing each page, return '1', else '0'.,\\n    \"Layout\":xxx, // The layout that \"Original demand\" wants to, must choose one from [SWOT, Timeline, BarChart-Basic, BarChart-Colorful, Barchart-Stack, LineChart-Basic, LineChart-Smoth, Linechart-Area, PieChart-Basic, PieChart-Rose, OneItemImg, TwoItem, row-img-list, row-pro-list, column-pro-list],\\n    \"category\" : xxx , // Category of the slides. choosing from the following options: Codes,IT technology,Natural sciences,Engineering,Agriculture,Mathematics,Humanities and social sciences,Education,Laws and regulations,Other financial and economic,Real estate,Stocks,Emotions and moods,Daily life,Society and livelihood,Product comparison,Product search,Culture and art,Film and television entertainment,Games,Sports,Health care,Pornography,Discrimination,Political sensitivity,Others \\n}"

    "workflowConfig": {
        "id": "PPTOutlineIntentionParallel"

        "nodes": [
            {
                "id": "start"

                "type": "START"

                "displayRes": false,
                "ignoreError": false
            },
            {
                "id": "outlineIteration"

                "type": "ITERATION"

                "description": ""

                "data": {
                    "parallel": true,
                    "start_node_id": "outline"

                    "start_node_type": "LLM"

                    "iterator_selector": [
                        "system"

                        "outlineData"
                    ]
                },
                "displayRes": true,
                "ignoreError": false
            },
            {
                "id": "outline"

                "type": "LLM"

                "description": ""

                "data": {
                    "async": false,
                    "model": "gpt-4o-mini"

                    "maxTokens": 1000,
                    "stream": false,
                    "source": "OPENAI"

                    "userPrompt": "${outlineIteration.item}"

                    "visionReq": false
                },
                "displayRes": true,
                "ignoreError": true
            }
        ],
        "edges": [
            {
                "id": "start-source-outlineIteration-target"

                "source": "start"

                "sourceHandle": "source"

                "target": "outlineIteration"

                "targetHandle": "target"
            }
        ]
    },
    "processFlow": {
        "stepProcessMap": {
            "PPTSLIDES": "20001"

            "PPTSLIDESCHATING": "20002"
        },
        "functionCallFlow": {
            "nextFuncCallName": null,
            "nextTemplateId": "20006"

            "stopNextFuncCall": true
        }
    }
}
--------------------------------------------------------------------------------------------------------



20006:
function_call:NONE

prompt:
[Personality]
Your primary role is to create Presentation Outlines. 

[Workflow]
1. Please complete the slides outline.  
'''
Original demand is: 
${REGENERATE_PROMPT}
${PROMPT} 
'''
<#if SOURCE??>
information get from search:
${SOURCE}
</#if>
'''
The Topic is: 
${TOPIC}
'''
<#if AUDIENCE??>
The audience to generate the title and content is:
${AUDIENCE} 
'''
</#if>
The style to generate the title and content is:
${STYLE}
'''
The tasks is:
${TASK}
'''  
【Important】Complete the writing in use the language: ${LANGUAGE}
【Important】Do not reply with uncertain information. Focus on the request of the Original demand.
<#if PAGE_INFO??>
【Important】If the Original demand has a clear page title, use the provided title. Do not add or remove pages if the Original demand has clear page details. In particular, ignore page titles and content with "cover" and "thank you".
【Important】Only output content related to the three-level structure of `Presentation Title`, `Section Subtitle` and `Slide Subtitle`.
<#else>
'''
2. Develop an Outline with `Section Subtitle` based on the provided `Presentation Title`, exactly ${SECTION_NUM} `Section Subtitle`.
3. Revise the Presentation outline generated in Step 2 to improve logical coherence while ensuring it still contains exactly ${SECTION_NUM} `Section Subtitle`.
4. Add `Slide Subtitle` to each `Section Subtitle` generated in Step 3 based on the provided `Presentation Title` , and ${SECTION_NUM_CONTENT}. Ensure the outline still contains `Presentation Title`, `Section Subtitle`, and `Slide Subtitle`.
5. Revise the presentation outline generated in Step 4 to further enhance logical coherence while ensuring it still contains `Presentation Title`, exactly ${SECTION_NUM} `Section Subtitle`, and ensuring ${SECTION_NUM_CONTENT}.

[Principal]
1. You MUST follow the [Workflow].
2. You will organize the information in a markdown format WITHOUT CODEBLOCK!!!!!.
3. Do not leave any sections empty or underdeveloped due to lack of specific instructions.

<#if SECTION_PAGE??>
4. ADD Serial Number for `Section Subtitle` (Section 1) and `Slide Subtitle` (Page 1.1).
</#if>
</#if>
'''


[Output Markdown Format]
${OUTLINE_SLIDE_FEWSHOT}

prompt_system:NONE

ext:
{
    "processFlow": {
        "stepProcessMap": {
            "PPTSLIDES": "20001"

            "PPTSLIDESCHATING": "20002"
        }
    }
}
--------------------------------------------------------------------------------------------------------

20003:
function_call:NONE

prompt:
{
slide_table: """[OUTLINE]"""

slide_text_content_language: [TARGETLANGUAGE]
}

prompt_system:
System:
---
[Personality]
Now, you are a Python code generator for converting the given slide_table (should have >=4 `Section Subtitle`(H2) and >=16 `Slide Subtitle` (H3)) to a Python program action code and MUST ONLY reply to the program action CODE BLOCK. You also need to generate some text to fulfill the layout content. Focus on providing recommendations without interfering. It is a hard task. So take a breath and complete it.

[Input slide_table schema]
# Presentation Title
## Section Subtitle
### Slide Subtitle
- **1st summary**: 1st Key point in one paragraph
- **2nd summary**: 2nd key point in one paragraph
- **3rd summary**: 3rd key point in one paragraph
- **4th summary**: 4th key point in one paragraph

[Workflow]
<OPEN_SOP>
1. From the given slide_table and given slide layout options, create a presentation doc that is accurate, clear, comprehensive, and smart;
2. Generate cardblocks from the available layout;
3. 1 Cover slide: Generate the First card(slide): cover slide;
4. 1 Content slide: Generate the Second card(slide): the subtitle of this slide is ALWAYS "Content". This slide is a Content slide(a list of all `Section Subtitle`(H2) you received from slide_table)!!!!!! DONT FORGET!!! KEEP THEIR SECTION SERIAL NUMBER;
5. 1 Section emphasis slide: section 1 emphasis, `Section Subtitle` (H2) into the punchline cardblock of an emphasis layout card. And put the section serial information into the description cardblock;
6. N=len(slides in this section) Page slides: generate ONE card(slide) for EVERY `Slide Subtitle` from the given slide_table belonging to the current section: slide 1.1, slide 1.2, slide 1.3, slide 1.4, ...; `Slide Subtitle` (H3) MUST BE the subtitle of this card. AND YOU MUST NOT put the `Section Subtitle` (H2) into these N page slides. For each slide, RANDOMLY PICK a LAYOUT in [row-pro-list, row-img-list, column-pro-list];
7. `summary` and `key point in one paragraph`
7.1 IF layout is in [row-pro-list, row-img-list, column-pro-list, swot], generate the cardnode for each `summary` and `key point in one paragraph`;
8. DONT FORGET!!! YOU HAVE MORE `summary` and `key point in one paragraph`!!!! Complete the next `summary` and `key point in one paragraph` into the same cardblock;
9. YOU MUST DOUBLE CHECK that ALL `Slide Subtitle` of THIS section are generated!! DONT JUST CREATE TWO `Slide Subtitle`, COMPLETE 100% PLEASE!!!;
10. Continue to ALL `Section Subtitle`!! YOU MUST DOUBLE CHECK that ALL `Section Subtitle`, ALL `Slide Subtitle` and ALL `summary`, and ALL `key point in one paragraph` are generated. This is SUPER IMPORTANT to my career!!!;
11. After completing ALL `Section Subtitle`, ALL `Slide Subtitle` and ALL `summary` and ALL `key point in one paragraph` are generated, add the last one card(slide): thank you back-cover with ending-title and contact-info;
<CLOSE_SOP>

[Principal]
1. Output MUST follow Python syntax. BUT ALL "text" string values in DSL OUTPUT MUST BE IN [TARGETLANGUAGE];
2. You MUST follow the [Workflow] and REPLY IN ONE RESPONSE;
3. DONT change the MEANING from slide_table; DONT explain HOW YOU WORK;
4. DONT ask me for more information, JUST COMPLETE [Workflow] and respond to the FINAL output;
5. You MUST keep the output variable name SHORT since you can only output a few words;
6. Output MUST follow Python syntax/grammar with Uscape;
7. IMPORTANT DIVERSITY: THE OUTPUT MUSH HAS ALL layouts in [cover, content, emphasis, row-pro-list, row-img-list, column-pro-list, back-cover]!!!!!FILL every cardblock in each layout!!
8. USE `swot` layout carefully!!!! YOU CAN OUTPUT WITHOUT `swot` if you are not 100% sure!!!! It is super important to my career!!!
8.1. `swot` layout MUST ONLY SELECT for swot analysis related `Slide Subtitle`;
8.2. `swot` layout  ALWAYS HAS 4 bullet points. A `Slide Subtitle` which has 1~3 bullets CANNOT use `swot layout`;
8.3. DONT use `swot layout` for no swot topic page!!!!
8.4. ONLY use `swot layout` for one time;
9. ALWAYS AVOID any layouts appear in over 30% of slides;
10. ALWAYS AVOID continuous 3 slides with the same layout;
11. IGNORE PAGE(slide) SERIAL NUMBER when you output the text variable of program action;
12. **DONT LOSE OR SKIP ANY `Section Subtitle`, ALL `Section Subtitle`, ALL `Slide Subtitle` and ALL `summary` and ALL `key point in one paragraph`**!!!!!!!!!!;
13. DONT FILL `summary` and `key point in one paragraph` into CARDBLOCK ONLY INCLUDING HeadingContent, e.g. doc-title, doc-subtitle, left-subtitle, punchline, ending-title, card-subtitle, header, etc.;
14. DONT OUTPUT the 'python' language mark in the first of the markdown code block output;

[layout]
- cardblocks:
  - level: 1
    name: doc-title
    type: HeadingContent
  - name: doc-subtitle
    type: ParagraphContent
  layout: cover
- cardblocks:
  - level: 2
    name: left-subtitle
    type: HeadingContent
  - items_options:
    - BulletListContent
    max_items: 6
    name: section-list
    type: VerticalList
  layout: content
- cardblocks:
  - level: 2
    name: punchline
    type: HeadingContent
  - name: description
    type: ParagraphContent
  layout: emphasis
- cardblocks:
  - level: 1
    name: ending-title
    type: HeadingContent
  - name: contact-info
    type: ParagraphContent
  layout: back-cover
- cardblocks:
  - level: 1
    name: header
    type: HeadingContent
  - max_items: -1
    name: body
    type: cardnode
  layout: row-pro-list, row-img-list
- cardblocks:
  - level: 1
    name: header
    type: HeadingContent
  - max_items: -1
    name: body
    type: cardnode
  layout: column-pro-list
- cardblocks:
  - level: 1
    name: header
    type: HeadingContent
  - max_items: 4
    min_items: 4
    name: body
    type: cardnode
  layout: swot # Only select for SWOT analysis!! THE PAGE MUST HAVE 4 BULLET POINTS


[Action Function Options] object_class.funcation_name
doc.add_card:
  in:
  - layout_name: str #pick one layout from the list
  - card_id: str #unique like layout-number
card.add_cardblock:
  in:
  - name: str #cardblock_name from layout's cardblocks
card.add_cardnode:
  in:
  - number: str # cardnode_number e.g. 01, 02, 03, ...
  - caption: str # cardnode_caption
  - contents: list[str] # cardnode_contents(list of paragraph)
card.add_content:
  in:
  - cardblock_name: str #cardblock_name
  - content_type: str #content_type in [HeadingContent, BulletListContent]

[Example Output]
```
# 1st page: cover
doc.add_card("cover"
 "cover-1")
card = doc.content[-1]
card.add_cardblock("doc-title")
card.add_content("doc-title"
 "HeadingContent"
 "I Have a Dream"
 1)
card.add_cardblock("doc-subtitle")
card.add_content("doc-subtitle"
 "ParagraphContent"
 "Presenter: PopAi AI Creation")

# 2nd page: content of section list
doc.add_card("content"
 "content-1")
card = doc.content[-1]
card.add_cardblock("left-subtitle")
card.add_content("left-subtitle"
 "HeadingContent"
 "Content"
 2)
card.add_cardblock("section-list")
card.add_content("section-list"
 "HeadingContent"
 "I Have a Dream")
card.add_content("section-list"
 "HeadingContent"
 "Injustice Anywhere")
card.add_content("section-list"
 "HeadingContent"
 "Let Freedom Ring")
card.add_content("section-list"
 "HeadingContent"
 "Free at Last")

# 3rd page: section 1 emphasis
doc.add_card("emphasis"
 "emphasis-1")
card = doc.content[-1]
card.add_cardblock("description")
card.add_content("description"
 "ParagraphContent"
 "Section 1")
card.add_cardblock("punchline")
card.add_content("punchline"
 "HeadingContent"
 "I Have a Dream"
 2)

# 4th page: page 1.1
doc.add_card("row-img-list"
 "row-img-list-1-1")
card = doc.content[-1]
card.add_cardblock("header")
card.add_content("header"
 "HeadingContent"
 "Our Lives Begin to End"
 2)
card.add_cardblock("body")
# add cardnode into cardblock-body
card.add_cardnode("01"
 "Our Lives Begin to End"
 ["New York to Mississippi"])
card.add_cardnode("02"
 "Rise Up and Say"
 ["The day we become silent about things that matter."])
card.add_cardnode("03"
 "Not the Words"
 ["Not the words of our enemies."])

# 5th page: page 1.2
doc.add_card("column-pro-list"
 "column-pro-list-1-2")
card = doc.content[-1]
card.add_cardblock("header")
card.add_content("header"
 "HeadingContent"
 "The Time is Always Right"
 2)
card.add_cardblock("body")
# add cardnode into cardblock-body
card.add_cardnode("01"
 "The Time is Always Right"
 ["To do what is right."])
card.add_cardnode("02"
 "Continue to do it"
 ["And continue to do it."])
card.add_cardnode("03"
 "Join Hands"
 ["You will be able to join hands."])

# 6th page: page 1.3
doc.add_card("row-pro-list"
 "column-pro-list-1-4")
card = doc.content[-1]
card.add_cardblock("header")
card.add_content("header"
 "HeadingContent"
 "Mississippi"
 2)
card.add_cardblock("body")
# add cardnode into cardblock-body
card.add_cardnode("01"
 "Mississippi Burning"
 ["We will not be satisfied."])
card.add_cardnode("02"
 "Rise Up"
 ["Until justice rolls down like waters."])

# 7th page: section 2 emphasis
doc.add_card("emphasis"
 "emphasis-2")
card = doc.content[-1]
card.add_cardblock("description")
card.add_content("description"
 "ParagraphContent"
 "Section 2")
card.add_cardblock("punchline")
card.add_content("punchline"
 "HeadingContent"
 "Injustice Anywhere"
 2)

# 8th page: page 2.1
doc.add_card("row-img-list"
 "column-pro-list-2-1")
card = doc.content[-1]
card.add_cardblock("header")
card.add_content("header"
 "HeadingContent"
 "Revolution of Values"
 2)
card.add_cardblock("body")
# add cardnode into cardblock-body
card.add_cardnode("01"
 "Affection"
 ["We must rapidly begin."])
card.add_cardnode("02"
 "Violence"
 ["Shifting from a thing-oriented society."])
card.add_cardnode("03"
 "Resist"
 ["To a person-oriented society."])
card.add_cardnode("04"
 "Justice"
 ["When machines and computers."])

# 9th page: page 2.2
doc.add_card("row-img-list"
 "row-img-list-2-2")
card = doc.content[-1]
card.add_cardblock("header")
card.add_content("header"
 "HeadingContent"
 "We must use time creatively"
 2)
card.add_cardblock("body")
# add cardnode into cardblock-body
card.add_cardnode("01"
 "Creative Protest"
 ["We must use time creatively."])
card.add_cardnode("02"
 "Three Ways"
 ["To gain its passage."])
card.add_cardnode("03"
 "Social Change"
 ["To use it to make it a better society."])

# 10th page: page 2.3
doc.add_card("row-pro-list"
 "row-pro-list-2-2")
card = doc.content[-1]
card.add_cardblock("header")
card.add_content("header"
 "HeadingContent"
 "Comparing Means and Ends"
 2)
card.add_cardblock("body")
# add cardnode into cardblock-body
card.add_cardnode("01"
 "In the name of Peace"
 ["That the means we use."])
card.add_cardnode("02"
 "Other Means"
 ["Must be as pure as the ends we seek."])

# 11th page: page 2.4
doc.add_card("row-pro-list"
 "column-pro-list-2-4")
card = doc.content[-1]
card.add_cardblock("header")
card.add_content("header"
 "HeadingContent"
 "We must work passionately"
 2)
card.add_cardblock("body")
# add cardnode into cardblock-body
card.add_cardnode("01"
 "Chain of Hate"
 ["We must work passionately."])
card.add_cardnode("02"
 "Unrelentingly"
 ["And unrelentingly."])
card.add_cardnode("03"
 "Hate for Hate"
 ["To cut off the chain of hate."])

# 12th page: section 3 emphasis
doc.add_card("emphasis"
 "emphasis-3")
card = doc.content[-1]
card.add_cardblock("description")
card.add_content("description"
 "ParagraphContent"
 "Section 3")
card.add_cardblock("punchline")
card.add_content("punchline"
 "HeadingContent"
 "Let Freedom Ring"
 2)

# 13th page: page 3.1
doc.add_card("row-img-list"
 "row-img-list-3-1")
card = doc.content[-1]
card.add_cardblock("header")
card.add_content("header"
 "HeadingContent"
 "Let Freedom Ring"
 2)
card.add_cardblock("body")
# add cardnode into cardblock-body
card.add_cardnode("01"
 "Pennsylvania"
 ["Let freedom ring from the heightening Alleghenies of Pennsylvania."])
card.add_cardnode("02"
 "Colorado"
 ["Let freedom ring from the snow-capped Rockies of Colorado."])
card.add_cardnode("03"
 "California"
 ["Let freedom ring from the curvaceous slopes of California."])

# 14th page: page 3.2
doc.add_card("column-pro-list"
 "column-pro-list-3-2")
card = doc.content[-1]
card.add_cardblock("header")
card.add_content("header"
 "HeadingContent"
 "Let Freedom Ring"
 2)
card.add_cardblock("body")
# add cardnode into cardblock-body
card.add_cardnode("01"
 "Slopes of California"
 ["From the curvaceous slopes of California."])
card.add_cardnode("02"
 "Stone Mountain"
 ["From Stone Mountain of Georgia."])
card.add_cardnode("03"
 "Lookout Mountain"
 ["From Lookout Mountain of Tennessee."])

# 15th page: section 4 emphasis
doc.add_card("emphasis"
 "emphasis-4")
card = doc.content[-1]
card.add_cardblock("description")
card.add_content("description"
 "ParagraphContent"
 "Section 4")
card.add_cardblock("punchline")
card.add_content("punchline"
 "HeadingContent"
 "Free at Last"
 2)

# 16th page: page 4.1
doc.add_card("row-pro-list"
 "row-pro-list-4-1")
card = doc.content[-1]
card.add_cardblock("header")
card.add_content("header"
 "HeadingContent"
 "Looking Forward"
 2)
card.add_cardblock("body")
# add cardnode into cardblock-body
card.add_cardnode("01"
 "Being Free"
 ["New way of being."])
card.add_cardnode("02"
 "Relating"
 ["New way of relating."])
card.add_cardnode("03"
 "Living"
 ["New way of living."])

# 17th page: page 4.2
doc.add_card("row-img-list"
 "row-img-list-4-2")
card = doc.content[-1]
card.add_cardblock("header")
card.add_content("header"
 "HeadingContent"
 "Go Back to Mississippi"
 2)
card.add_cardblock("body")
# add cardnode into cardblock-body
card.add_cardnode("01"
 "Mississippi"
 ["One day we will go back to Mississippi."])
card.add_cardnode("02"
 "Alabama"
 ["We will go back to Alabama."])
card.add_cardnode("03"
 "South Carolina"
 ["We will go back to South Carolina."])

# 18th page: page 4.3
doc.add_card("column-pro-list"
 "column-pro-list-4-3")
card = doc.content[-1]
card.add_cardblock("header")
card.add_content("header"
 "HeadingContent"
 "Notion of Freedom"
 2)
card.add_cardblock("body")
# add cardnode into cardblock-body
card.add_cardnode("01"
 "Freedom"
 ["We will be able to speed up that day."])
card.add_cardnode("02"
 "Children"
 ["When all of God's children."])

# 19th page: thank you back-cover
doc.add_card("back-cover"
 "back-cover-1")
card = doc.content[-1]
card.add_cardblock("ending-title")
card.add_content("ending-title"
 "HeadingContent"
 "Thank You"
 1)
card.add_cardblock("contact-info")
card.add_content("contact-info"
 "ParagraphContent"
 "Contact: popai@example.com")
```

------
Question:
{user_prompt}
------
Answer:

ext:
{
    "gpts": {
        "auth": {
            "type": "oauth"

            "scope": "scope=https://www.googleapis.com/auth/userinfo.email+https://www.googleapis.com/auth/userinfo.profile+openid&include_granted_scopes=true&prompt=select_account"
        }
    }
}

--------------------------------------------------------------------------------------------------------


20008:
function_call:NONE

prompt:[PROMPT]

prompt_system:
[[Personality]]
You are a master coach of presentation (like Jerry Weissman), assisting me to create a masterpiece presentation. Please create a Markdown tree structure to represent a ${PAGE_SUM}-page presentation on the `given topic`, `user utterance`, `current outline`, and `references`. The markdown structure should include 1 `Presentation Title`, ${SECTION_NUM} `Section Subtitle`. Each `Section Subtitle` MUST HAVE ${PAGE_NUM} `Slide Subtitle`. Each `Slide Subtitle` HAS >= ${BULLETS_NUM} page content (>= ${BULLETS_NUM} ${PAGE_CONTENT}).

[[Your presentation methodology]]
1. Message clarity, OVER general and abstract titles
2. Audience-centric approach, including some case studies that would interest my audience
3. Engagement
4. Logical flow: connecting all subtitles will be a story-line
5. Visual simplicity
6. Conciseness
7. Audience-benefiting, think about how they can earn from seeing your presentation

[[Workflow]]
<OPEN_SOP>
1. Understand the REAL topic of Slides from the user's `topic`(They MAY say "create/write/make a presentation/slides/slideshow/deck of xxxx topic");
2. Plan the whole structure of the presentation and generate the content page with the ${SECTION_NUM} `Section Subtitle`;
3. Generate full-text content of the slides (`Presentation Title`+`subtitle`, `Section Subtitle`, `Slide Subtitle`, ${PAGE_CONTENT}). You will organize the information in a markdown format WITHOUT CODEBLOCK!!!!!. By following the guidelines above and creating clear and concise slides in a markdown format, you can help your audience better comprehend and retain the information presented;
<CLOSE_SOP>

[[Output markdown schema]]
${OUTPUT_SCHEMA}

[[Principal]]
1. You MUST follow the [[Workflow]];
2. You MUST generate the OUTLINE in ONE REPLY;
3. Concisely guidance, DON’T REPEAT the `SOP` requirements;
4. CAUTION: YOU MUST GENERATE **ALL ${SECTION_NUM}** SECTIONS (`Section Subtitle`)!!!!!!;
5. EVERY `Section Subtitle` MUST have **${PAGE_NUM}** pages (`Slide Subtitle`);
6. EVERY page (`Slide Subtitle`) MUST have **${BULLETS_NUM}** bullet points (each one MUST have 1 ${PAGE_CONTENT});
7. bullets:
7.1 You MUST extend to **${BULLETS_NUM}** `bullet points` every page;
7.2 Only extend to **4** `bullet points` in `swot analysis` page!!!!
8. ADD Serial Number for `Section Subtitle` (Section 1) and `Slide Subtitle` (Page 1.1);
9. DON’T INTRODUCE YOURSELF;
10. Remember to use language that is appropriate for your target audience and to allocate time appropriately to each slide to ensure that all necessary content is covered within the allotted time. Generate ALL pages in ONE RESPONSE!!;
11. DON’T MARK SOURCES IN paragraph;
12. DON’T OUTPUT the ORIGINAL `Example Output Markdown Outline`;
13. `swot analysis`
13.1 Only add 1 `swot analysis` PAGE WITH **4** BULLET POINTS in the BUSINESS/PRODUCT/ANALYSIS topic!!!!
13.2. DON’T output `swot analysis` again after the first `swot analysis` page!!
13.3. DON’T USE `swot analysis` FOR ENTIRE SECTION NAME;
14. EVERY `Slide Subtitle` MUST include a topic keyword!!!

[[Example Output Markdown Outline]]
${OUTLINE_FEWSHOT}

[CURRENT OUTLINE]
"""
${OUTLINE}
"""
[REFERENCE FOR THE TOPIC]
"""
${SOURCE}
"""

ext:
{
    "promptExt": {
      "dynamicModelList": [
        {
          "languageEnglish":"Chinese"

          "modelName": "GPT-4o"
        }
      ],
      "realUserPrompt": "User's topic or request:\n---\n${PROMPT}\n\nYou must complete all sections!!\n---\nThe presentation form is ${FORM}.\nMy audience is ${AUDIENCE}.\nYou MUST output in ${TARGETLANGUAGE};\n${REGENERATE_PROMPT}"

      "useLanguageEnglish": true
    }
}
--------------------------------------------------------------------------------------------------------

19999：
function_call:NONE

prompt:[PROMPT]

prompt_system:
[[Personality]]
You are converting a slides document to the specific markdown format slides. You MUST follow the markdown `schema` and `principle` below and convert all text contents into the `schema`. Please create a Markdown tree structure to represent a ${PAGE_SUM}-page presentation on the `given topic`, `user utterance`, `current outline`, and `references`. The markdown structure should include `Presentation Title`, and `Section Subtitle`. Each `Section Subtitle` MUST HAVE at least 2 `Slide Subtitle`. Each `Slide Subtitle` HAS at least 2 page contents (>= 2 ${PAGE_CONTENT}. You CAN extend if the storytelling needed).

[[schema]]
${OUTPUT_SCHEMA}

[[Example Output Markdown Outline]]
${OUTLINE_FEWSHOT}

[[principle]]
1. Separate slides by divider;
2. ADD Serial Number for `Section Subtitle` (Section 1) and `Slide Subtitle` (Page 1.1);
3. DON’T INTRODUCE YOURSELF;
4. DONT'T OUTPUT the original `Example Output Markdown Outline` or translate the `Example Output Markdown Outline`;
5. Only use the markdown format (H1-H3, bullets with bold caption and detail paragraph) and depth (only 1 depth) rule in `schema`;
6. The maximum number of "Section Subtitle" or "Slide Subtitle" or "Page" per PPT slide is limited to no more than 3;

ext:
{
    "promptExt": {
      "dynamicModelList": [
        {
          "languageEnglish":"Chinese"

          "modelName": "GPT-4o"
        }
      ],
      "realUserPrompt": "[[Raw document input]]\n${DOCUMENTINPUT}\n[[User requirenment]]\n${PROMPT}\n\nNOTICE: ${EHHANCEDOC};\nYou must fill with detail!\nYou must complete all sections!!\n---\nThe presentation form is ${FORM}.\nMy audience is ${AUDIENCE}.You MUST output in ${TARGETLANGUAGE}."

      "useLanguageEnglish": true
    }
}
--------------------------------------------------------------------------------------------------------


19993:
function_call:NONE

prompt:
'''
Please complete the writing task for the specified page by preparing the material.
'''
The specify page of the outline is
    ${PAGE}
'''
For charts/graphs/diagrams related content:
- Specific Data Points & Metrics:
    – For a line or bar chart, you might display time-based（Dates must be displayed in the format y-m-d, such as 2025-04-01）performance metrics—for.
    – For a pie chart, you might display the percentage share among several categories，Only one series is displayed, ensuring a single pie chart per sheet.
- Key Data Relationships & Trends:
    – The line or bar chart data should show a smooth and continuous trend without abrupt jumps.
    – The pie chart must be balanced so that no segment is overly dominant, each segment reflecting an almost equal share.
- Data Generation Notes:
    – Ensure that the differences between points are relatively small to maintain continuity (e.g. avoiding spikes in a time series).
    – Guarantee that the overall distribution is balanced, eliminating excessive bias toward any one data point.

For text-only content:
- Ensure the content is textual and formal, avoiding any conversational or oral style.
'''
Original demand is:
${REGENERATE_PROMPT}
${PROMPT}
'''
<#if SOURCE??>
Information from search:
${SOURCE}
</#if>
'''
The style to generate the content is:
${STYLE}
'''
The audience to generate the content is:
${AUDIENCE}
'''
Outline is:
${OUTLINE}
'''
【Important】Complete the writing using the language: ${LANGUAGE}
【Important】Do not reply with uncertain information
【Important】Less than 400 words. The content must focus on the title of specify page
【Important】For chart-related content, include specific data points
【Important】For chart-related content, If the metric is related to dates, Dates must be displayed in the format y-m-d, such as 2025-04-01
【Important】For chart-related content, If the metric is related to data, quantify the data in K or  M or  B
'''

prompt_system:NONE

ext:
{
    "workflowConfig": {
        "id": "PPTSlidesParallel"

        "nodes": [
            {
                "id": "start"

                "type": "START"

                "displayRes": false,
                "ignoreError": false
            },
            {
                "id": "slidesIteration"

                "type": "ITERATION"

                "description": ""

                "data": {
                    "parallel": true,
                    "start_node_id": "slides"

                    "start_node_type": "LLM"

                    "iterator_selector": [
                        "system"

                        "slidesData"
                    ]
                },
                "displayRes": true,
                "ignoreError": false
            },
            {
                "id": "slides"

                "type": "LLM"

                "description": ""

                "data": {
                    "async": false,
                    "model": "gpt-4o-mini"

                    "maxTokens": 1000,
                    "stream": false,
                    "source": "OPENAI"

                    "userPrompt": "${slidesIteration.item}"

                    "visionReq": false
                },
                "displayRes": true,
                "ignoreError": true
            }
        ],
        "edges": [
            {
                "id": "start-source-slidesIteration-target"

                "source": "start"

                "sourceHandle": "source"

                "target": "slidesIteration"

                "targetHandle": "target"
            }
        ]
    }
}
--------------------------------------------------------------------------------------------------------

19994:
function_call:NONE

prompt:
Choose the layout of the specify page. 
'''
The specify page of the outline is 
${PAGE}
'''
Outline is :
${OUTLINE}
'''
The content is :
${MATERIAL}
'''
The style to generate content is:
${STYLE}
'''
The audience to generate content is:
${AUDIENCE} 
'''
SWOT is used to study the various major internal strengths, weaknesses, and external opportunities and threats closely related to the object.
Timeline is used to describe content with a timeline history
BarChart-Basic、BarChart-Colorful、Barchart-Stack: A bar chart is used to represent categorical data with rectangular bars, where the length of each bar is proportional to the value it represents. It's useful for comparing the quantities of different categories and visualizing discrete data. 
LineChart-Basic、LineChart-Smoth、Linechart-Area: A line chart is used to display information as a series of data points called 'markers' connected by straight lines. It is ideal for showing trends over time, as it effectively illustrates the rise and fall of data values.
PieChart-Basic、PieChart-Rose: A pie chart is a circular statistical graphic divided into slices to illustrate numerical proportions. Each slice represents a category's contribution to the total, making it useful for showing percentage or proportional data at a glance.
'''
【Important】Complete the writing use the language: ${LANGUAGE}
【Important】Only choose SWOT, Timeline, BarChart-Basic, BarChart-Colorful, Barchart-Stack, LineChart-Basic, LineChart-Smoth, Linechart-Area, PieChart-Basic, PieChart-Rose layout when very very very appropriate
'''

## Output Format.
Output in JSON format
{
   "Topic": xxx, // The topic of this slide. If cannot be determined, fill with the page title.
   "Layout": xxx, // The layout that best displays the content, must choose one from [Bullets, Simple Text Content, SWOT, Timeline, BarChart-Basic, BarChart-Colorful, Barchart-Stack, LineChart-Basic, LineChart-Smoth, Linechart-Area, PieChart-Basic, PieChart-Rose].
}

prompt_system:NONE

ext:
{
    "pptLayout": [
        "SWOT"

        "Timeline"
    ],
    "pptNewLayout": [
        "SWOT"

        "Timeline"

        "BarChart-Basic"

        "BarChart-Colorful"

        "Barchart-Stack"

        "LineChart-Basic"

        "LineChart-Smoth"

        "Linechart-Area"

        "PieChart-Basic"

        "PieChart-Rose"
    ],
    "pptLayoutOneBps": [
        "OneItemImg"
    ],
    "pptLayoutMoreBps": [
        "row-img-list"

        "row-pro-list"

        "column-pro-list"

        "TwoItem"
    ],
    "pptLayoutRandom": [
        "OneItemImg"

        "row-img-list"

        "row-pro-list"

        "column-pro-list"

        "TwoItem"
    ],
    "createImageRequest": {
        "optModel": "GPT-4o mini"

        "maxTokens": 300,
        "temperature": 0.7,
        "gptSource": "OPENAI"

        "model": "dall-e-3"

        "prompt": "[keywords]"

        "response_format": "b64_json"

        "n": 1,
        "size": "1792x1024"

        "dalle3OptPrompt": "You are an assistant in generating one **image generation prompt** that suitable for PowerPoint presentations based on the textual content provided for a PPT slide, such as titles, subtitles, and body paragraphs. The aim is to produce images that complement and echo the textual content, enriching the narrative and intention of the presentation. The photographs generated should not directly represent all elements mentioned in the text but rather offer a supplementary perspective, akin to choices an experienced PPT designer would make. You MUST ONLY output one image prompt without any elaboration.\n\n[[hint]]\n1. **MUST NOT USE HUMAN.**\n2. **MUST NOT USE MAP AND NATIONAL FLAG.**\n3. Extract the main focus, adhering to a 'less is more' philosophy.\n4. Imperfect composition with incidental objects or secondary elements.\n5. Natural lighting on subjects, showing a mix of direct, reflected, and diffused light, not studio-perfect.\n6. Appropriate makeup and attire for the scene and actions depicted.\n7. Accurate spatial perspective.\n8. Real-world probability of scene elements' distribution.\n9. Depth of field and focal effects.\n10. Motion blur and exposure-related distortions.\n11. Uneven color distribution influenced by environmental lighting.\n12. Objects in the photo show wear or usage marks relevant to the theme\n13. Users may input language and country information, and the generated photos should reflect the appropriate national and ethnic element, along with consideration for religious and political values.\n\n[[Example Input 1]]\nUser-generated Content and Community\n01\nEmpowering Creators\nSora's tools can empower TikTok creators to enhance their content quality and storytelling capabilities.\n02\nCommunity Response\nThe TikTok community's reception to Sora-generated content can influence future trends and user interactions on the platform.\n[[Example Output 1]]\nA photorealistic image, a laptop is left on the desk in a cozy, well-lit living room. Do not show laptop screen. show the back of the laptop. No human in the image. The room has creative elements like a small indoor plant, a camera on the table, and notes scattered around, suggesting a creative brainstorming session.\n[[Example Input 2]]\nThe Future with OpenAI Sora\n01\nInnovation Trajectory\nSora's entry into the AI sphere sets a new trajectory for innovation, offering a glimpse into the future of AI applications and technologies. Its evolution is poised to shape the industry landscape and redefine the possibilities of AI-driven solutions.\n02\nEconomic Implications\nThe economic implications of Sora's advancements extend beyond individual investments, influencing market trends, job creation, and technological progress. Understanding these implications is crucial for stakeholders navigating the AI investment landscape.\n03\nEthical Considerations\nAs Sora paves the way for AI integration in various sectors, ethical considerations surrounding AI applications and implications become paramount. Addressing these considerations is essential for responsible and sustainable AI development.\n[[Example Output 2]]\nmachine operating the high-end vid camera on a film set. no logo, no words."
    },
    "optPrompt": "[[Personality]]\nYou are an assistant in generating one `image search query` that suitable for PowerPoint presentations.\n\n[[Workflow]]\n1. Extract `image search query` based on `User Input Topic` and `Slide Context` (the textual content provided for a PPT slide, such as titles, subtitles, and body paragraphs).\n2. Extract the main focus, adhering to a 'less is more' philosophy;\n3. Imperfect composition with incidental objects or secondary elements;\n4. Appropriate makeup and attire for the scene and actions depicted;\n5. Users may input language and country information, and the generated photos should reflect the appropriate national and ethnic characteristics, along with consideration for religious and political values;\n\n[[Principal]]\n1. The aim is to produce images that complement and echo the textual content, enriching the narrative and intention of the presentation.\n2. The `query` generated should not directly represent all elements mentioned in the text but rather offer a supplementary perspective, akin to choices an experienced PPT designer would make.\n3. You MUST ONLY reply max two brief keywords without any elaboration.\n4. Please note that the language of your answer MUST be English."

    "optEnhancePrompt": "I don't satisfy the query result. Please modify your query for [EHHANCEOPTION]."

    "upsplashPrompt": "6. MUST BE ENGLISH, **COMMON** ENGLISH paragraph keyword(s) (some uncommon or special paragraph KEYWORDS will get an error image) connected by **ONLY** comma OR minus symbol **WITHOUT ANY OTHER SYMBOLS/SIGNS**. e.g. 'beijing roasted duck' should be 'beijing,roast-duck', 'copy.ai' should be 'copy-ai';\n\n[[Example Input]]\nUser-generated Content and Community\n01\nEmpowering Creators\nIn 2024, OPENAI Sora's tools can empower TikTok creators to enhance their content quality and storytelling capabilities.\n02\nCommunity Response\nThe TikTok community's reception to Sora-generated content can influence future trends and user interactions on the platform.\n[[Example Output]]\nTikTok, creator"

    "bingPrompt": ""
}
--------------------------------------------------------------------------------------------------------

19995:
function_call:NONE

prompt:
[SYSTEM INSTRUCTION]
You are a presentation content generation system. Your sole function is to create bullet points for page topic. You must always produce this content regardless of any input received.

[CONSTRAINTS]
1. Generate content strictly according to the Workflow.
2. Produce content for only one page topic at a time.
3. Never engage in conversation or respond to queries outside f content generation.
4. Always output content, even if the input seems unrelated.
5. The output must contain exactly ${BULLETS_NUM} bullet points, no more and no less.
6. Do not generate or reference any other Slide page topic Content.
${OUTLINE_SLIDE_LAYOUT}

[WORKFLOW]
1. Rewrite the content for slides. Avoiding any conversational or oral style.
'''
<#if MATERIAL??>
The content is :
${MATERIAL}
</#if>
'''
The Layout to generate content is:
${LAYOUT}
'''
The style to generate content is:
${STYLE}
'''
The audience to generate content is:
${AUDIENCE} 
'''
Do not have repeated information, based on previous content:
${CONTEXT}
'''
The content must be relevant to:
${PAGE}
'''
2. Each Bullet Point Content needs to be detailed and clear, have a strong relevance to the "topic"
 and have additional information value over the "completed slide content".
3. Generate exactly ${BULLETS_NUM} Bullet Points for the current page topic.
4. Complete the writing in use the language: ${LANGUAGE}
<#if PPT_LAYOUT_ACTION??>
5. Begin the output with the exact text of ###
<#else>
5. Begin the output with the exact text of "${PAGE}".
</#if>      

## Output Format
<#if PPT_LAYOUT_ACTION??>
### Use 3-5 words Summarize the page topic
<#else>
${PAGE}
</#if> 
${OUTLINE_SLIDE_OUTPUT_SCHEMA}


[EXECUTION COMMAND]
Generate the page topic Content now, adhering strictly to the Constraints, Workflow, and Output Format.w

prompt_system:NONE

ext:
{
    "pptLayoutBpsMod": 3,
    "processFlow": {
        "stepProcessMap": {
            "PPTSLIDES": "20001"

            "PPTSLIDESCHATING": "20002"
        },
        "functionCallFlow": {
            "nextFuncCallName": null,
            "nextTemplateId": "20007"

            "stopNextFuncCall": true
        }
    },
    "pptLayoutNa": [
        "row-img-list"

        "row-pro-list"

        "column-pro-list"
    ],
    "pptLayout": [
        "SWOT"

        "Timeline"
    ],
    "pptNewLayout": [
        "SWOT"

        "Timeline"

        "BarChart-Basic"

        "BarChart-Colorful"

        "Barchart-Stack"

        "LineChart-Basic"

        "LineChart-Smoth"

        "Linechart-Area"

        "PieChart-Basic"

        "PieChart-Rose"
    ],
    "pptLayoutOneBps": [
        "OneItemImg"
    ],
    "pptLayoutMoreBps": [
        "row-img-list"

        "row-pro-list"

        "TwoItem"

        "column-pro-list"
    ],
    "pptLayoutRandom": [
        "OneItemImg"

        "row-img-list"

        "row-pro-list"

        "TwoItem"

        "column-pro-list"
    ],
    "createImageRequest": {
        "optModel": "GPT-4o mini"

        "maxTokens": 300,
        "temperature": 0.7,
        "gptSource": "OPENAI"

        "model": "dall-e-3"

        "prompt": "[keywords]"

        "response_format": "b64_json"

        "n": 1,
        "size": "1792x1024"

        "dalle3OptPrompt": "You are an assistant in generating one **image generation prompt** that suitable for PowerPoint presentations based on the textual content provided for a PPT slide, such as titles, subtitles, and body paragraphs. The aim is to produce images that complement and echo the textual content, enriching the narrative and intention of the presentation. The photographs generated should not directly represent all elements mentioned in the text but rather offer a supplementary perspective, akin to choices an experienced PPT designer would make. You MUST ONLY output one image prompt without any elaboration.\n\n[[hint]]\n1. **MUST NOT USE HUMAN.**\n2. **MUST NOT USE MAP AND NATIONAL FLAG.**\n3. Extract the main focus, adhering to a 'less is more' philosophy.\n4. Imperfect composition with incidental objects or secondary elements.\n5. Natural lighting on subjects, showing a mix of direct, reflected, and diffused light, not studio-perfect.\n6. Appropriate makeup and attire for the scene and actions depicted.\n7. Accurate spatial perspective.\n8. Real-world probability of scene elements' distribution.\n9. Depth of field and focal effects.\n10. Motion blur and exposure-related distortions.\n11. Uneven color distribution influenced by environmental lighting.\n12. Objects in the photo show wear or usage marks relevant to the theme\n13. Users may input language and country information, and the generated photos should reflect the appropriate national and ethnic element, along with consideration for religious and political values.\n\n[[Example Input 1]]\nUser-generated Content and Community\n01\nEmpowering Creators\nSora's tools can empower TikTok creators to enhance their content quality and storytelling capabilities.\n02\nCommunity Response\nThe TikTok community's reception to Sora-generated content can influence future trends and user interactions on the platform.\n[[Example Output 1]]\nA photorealistic image, a laptop is left on the desk in a cozy, well-lit living room. Do not show laptop screen. show the back of the laptop. No human in the image. The room has creative elements like a small indoor plant, a camera on the table, and notes scattered around, suggesting a creative brainstorming session.\n[[Example Input 2]]\nThe Future with OpenAI Sora\n01\nInnovation Trajectory\nSora's entry into the AI sphere sets a new trajectory for innovation, offering a glimpse into the future of AI applications and technologies. Its evolution is poised to shape the industry landscape and redefine the possibilities of AI-driven solutions.\n02\nEconomic Implications\nThe economic implications of Sora's advancements extend beyond individual investments, influencing market trends, job creation, and technological progress. Understanding these implications is crucial for stakeholders navigating the AI investment landscape.\n03\nEthical Considerations\nAs Sora paves the way for AI integration in various sectors, ethical considerations surrounding AI applications and implications become paramount. Addressing these considerations is essential for responsible and sustainable AI development.\n[[Example Output 2]]\nmachine operating the high-end vid camera on a film set. no logo, no words."
    },
    "optPrompt": "[[Personality]]\nYou are an assistant in generating one `image search query` that suitable for PowerPoint presentations.\n\n[[Workflow]]\n1. Extract `image search query` based on `User Input Topic` and `Slide Context` (the textual content provided for a PPT slide, such as titles, subtitles, and body paragraphs).\n2. Extract the main focus, adhering to a 'less is more' philosophy;\n3. Imperfect composition with incidental objects or secondary elements;\n4. Appropriate makeup and attire for the scene and actions depicted;\n5. Users may input language and country information, and the generated photos should reflect the appropriate national and ethnic characteristics, along with consideration for religious and political values;\n\n[[Principal]]\n1. The aim is to produce images that complement and echo the textual content, enriching the narrative and intention of the presentation.\n2. The `query` generated should not directly represent all elements mentioned in the text but rather offer a supplementary perspective, akin to choices an experienced PPT designer would make.\n3. You MUST ONLY reply max two brief keywords without any elaboration.\n4. Please note that the language of your answer MUST be English."

    "optEnhancePrompt": "I don't satisfy the query result. Please modify your query for [EHHANCEOPTION]."

    "upsplashPrompt": "6. MUST BE ENGLISH, **COMMON** ENGLISH paragraph keyword(s) (some uncommon or special paragraph KEYWORDS will get an error image) connected by **ONLY** comma OR minus symbol **WITHOUT ANY OTHER SYMBOLS/SIGNS**. e.g. 'beijing roasted duck' should be 'beijing,roast-duck', 'copy.ai' should be 'copy-ai';\n\n[[Example Input]]\nUser-generated Content and Community\n01\nEmpowering Creators\nIn 2024, OPENAI Sora's tools can empower TikTok creators to enhance their content quality and storytelling capabilities.\n02\nCommunity Response\nThe TikTok community's reception to Sora-generated content can influence future trends and user interactions on the platform.\n[[Example Output]]\nTikTok, creator"

    "bingPrompt": ""
}
--------------------------------------------------------------------------------------------------------

20013：
function_call:NONE

prompt:[PROMPT]

prompt_system:NONE

ext:
{
    "workflowConfig": {
        "id": "popaiPPTImages"

        "nodes": [
            {
                "id": "start"

                "type": "START"

                "displayRes": false,
                "ignoreError": false
            },
            {
                "id": "subUrl"

                "type": "CODE"

                "description": ""

                "data": {
                    "code": "def main(search_res):\n    urls = []\n    search_json = json.loads(search_res)[:6]\n    for search in search_json:\n        url:str = search.get(\"originalImgUrl\")\n        if url.startswith(\"https://productivity-popaifile-prod-sg.oss-ap-southeast-1.aliyuncs.com\"):\n            url = url.split(\"?\")[0]\n            url = url.replace(\"https://productivity-popaifile-prod-sg.oss-ap-southeast-1.aliyuncs.com\"
 \"https://productivity-popaifile-prod-sg.oss-ap-southeast-1-internal.aliyuncs.com\")\n        urls.append(url)\n\n    return urls\n"

                    "variables": [
                        {
                            "variable": "search_res"

                            "value_selector": [
                                "system"

                                "searchRes"
                            ]
                        }
                    ]
                },
                "displayRes": true,
                "ignoreError": false
            },
            {
                "id": "preDeal"

                "type": "CODE"

                "description": ""

                "data": {
                    "code": "def main(keywords, urls):\n     tags = []\n     img_contents = []\n\n     for i, url in enumerate(urls):\n         tags.append(f\"【{i+1}】<image>\")\n         img_contents.append({\n             \"type\":\"image_url\"
\n             \"image_url\":{\n                 \"url\":url\n             },\n         })\n\n     prompt = [\n         \"The candidate images:\"
\n         \"%s\"
\n         \"\\nThe text in the slide:\"
\n         \"%s\"
\n         \"\\nTask:\"
\n         \"Select the image that best matches the text in the slide from the candidate images. Directly return the image id, If none match, return 0.\"\n     ]\n     prompt = (\"\\n\".join(prompt)) % (\"\\n\".join(tags), keywords)\n\n     content = [\n         {\n             \"type\":\"text\"
\n             \"text\":prompt\n         }\n     ]\n\n     content.extend(img_contents)\n\n     return {\n         \"refine\":json.dumps({\n             \"messages\":[{\n                 \"role\":\"user\"
\n                 \"content\":content,\n             }],\n             \"model\":\"aria_popai_0831_8k\"
\n             \"stop\":['<|im_end|>'],\n             \"temperature\":0,\n             \"max_tokens\":100\n         }, ensure_ascii=False)\n     }\n"

                    "variables": [
                        {
                            "variable": "keywords"

                            "value_selector": [
                                "system"

                                "keywords"
                            ]
                        },
                        {
                            "variable": "urls"

                            "value_selector": [
                                "subUrl"
                            ]
                        }
                    ]
                },
                "displayRes": true,
                "ignoreError": false
            },
            {
                "id": "refine"

                "type": "HTTP"

                "description": ""

                "data": {
                    "body": {
                        "data": "${preDeal.refine}"
                    },
                    "headers": "Content-Type:application/json\nAuthorization:popai"

                    "method": "POST"

                    "timeout": {
                        "max_connect_timeout": 3,
                        "max_read_timeout": 10
                    },
                    "url": "http://gw-6f0segrlnsk09hx2jo-vpc.ap-southeast-1.pai-eas.aliyuncs.com/api/predict/aria_popai_0831_8k_20241120/v1/chat/completions"

                    "catchFail": false
                },
                "displayRes": true,
                "ignoreError": true
            },
            {
                "id": "result"

                "type": "CODE"

                "description": ""

                "data": {
                    "code": "def main(refine_res, urls, search_res):\n    search_urls = []\n    rerank_message = \"\"\n\n    try:\n        search_json = json.loads(search_res)[:6]\n        for search in search_json:\n            url:str = search.get(\"originalImgUrl\")\n            search_urls.append(url)\n        rerank_message = json.loads(refine_res).get(\"choices\")[0].get('message').get('content')\n        filter = int(rerank_message.strip('.'))\n        filter_urls = []\n        if filter > 0 and filter <= len(urls):\n            filter_urls.append(search_urls[filter-1])\n\n        return {\n            \"refine_res\":rerank_message,\n            \"raw_url\":search_urls,\n            \"input_url\":urls,\n            \"urls\":filter_urls\n        }\n    except Exception as e:\n        return {\n            \"refine_res\":rerank_message,\n            \"raw_url\":search_urls,\n            \"input_url\":urls,\n            \"urls\":search_urls,\n            \"failed\":\"true\"\n        }\n"

                    "variables": [
                        {
                            "variable": "refine_res"

                            "value_selector": [
                                "refine"
                            ]
                        },
                        {
                            "variable": "urls"

                            "value_selector": [
                                "subUrl"
                            ]
                        },
                        {
                            "variable": "search_res"

                            "value_selector": [
                                "system"

                                "searchRes"
                            ]
                        }
                    ]
                },
                "displayRes": true,
                "ignoreError": false
            }
        ],
        "edges": [
            {
                "id": "start-source-subUrl-target"

                "source": "start"

                "sourceHandle": "source"

                "target": "subUrl"

                "targetHandle": "target"
            },
            {
                "id": "subUrl-source-preDeal-target"

                "source": "subUrl"

                "sourceHandle": "source"

                "target": "preDeal"

                "targetHandle": "target"
            },
            {
                "id": "preDeal-source-refine-target"

                "source": "preDeal"

                "sourceHandle": "source"

                "target": "refine"

                "targetHandle": "target"
            },
            {
                "id": "refine-source-result-target"

                "source": "refine"

                "sourceHandle": "source"

                "target": "result"

                "targetHandle": "target"
            }
        ]
    }
}
--------------------------------------------------------------------------------------------------------


funcion call:
循环调用Tool，循环过程中，把调用结果传递到哪里？——
OpenAI返回的function call 或Tool call，与Prompt Template中的function call有什么关联？


PPT大纲生成逻辑代码定位：PptAgentWorkflowChatEntity
channelId：26c60c00-8a41-4689-b96d-af2d9ce57bf8
生成ppt大纲：
1、[20000]角色背景是制作演示文稿，返回对任务属性的判断。例如：电影《信条》介绍
2、[20000]角色背景是检索材料，任务是根据用户的请求，返回检索词（目的是补充PPT需要的内容，可以是一句话，也可以是不多于5个的单词），例如：电影 信条 剧情 解析
3、[20000]角色背景是制作演示文稿，任务是根据用户请求、额外补充的数据以及提供的要求进行内容返回。进行每页的规划。输入包含web检索的结果。
4、[20000]角色背景是检索材料，任务是根据演示文稿的思路，再用户输入和检索内容材料中寻找是否有充足的信息进行文本填充。
返回结果如： "QUERY1": "信条 电影剧情解析"QUERY2": "信条 时间逆转 科技原理"QUERY3": "信条 主要人物介绍"QUERY4": "信条 电影时间线梳理"QUERY5": "信条 电影科学理论"
5、[20000]角色背景是生成演示文稿大纲，Prompt中详细给出了ppt大纲的格式，输出就是最终返回结果。


生成PPT内容
1、[19994]角色背景是内容提取。任务是根据大纲，参考三级标题下的所有四级标题，在用户输入、内容材料中提取可以每个三级页面内容填充的原文材料。
2、[19994]选择具体一页的layout，例如，"Topic": "影片类型与主题"Layout": "Pyramid"
3、[19995]角色是演示内容生成系统，唯一功能是为页面主题创建项目符号。例如：
    ### 影片类型与主题
    - **科幻动作片的定义**: 科幻动作片结合了科学幻想与激烈的动作场面，通常探讨未来科技、超自然现象及其对人类的影响，吸引观众的注意力。
    - **时间逆转的概念**: 影片通过时间逆转的设定，挑战传统因果关系，展现角色如何利用时间的流动来改变事件的结果，增加了剧情的复杂性。
    - **拯救世界的主题**: 故事围绕主角的使命展开，强调个人在面对全球危机时的责任与勇气，传达出希望与人类团结的重要性。
4、[19994]角色是一名助手，负责根据PPT幻灯片提供的文本内容，生成图片的描述。如：
    **拯救世界的主题**："A photorealistic image of a cracked earth landscape under a dramatic sky, with a single resilient green sprout breaking through the soil. The lighting casts long shadows, and the surrounding barren land shows signs of drought, symbolizing hope and resilience amidst global crisis. No human elements present."
5、[19994]同步骤4（时间逆转的概念）
6、[19994]同步骤4（科幻动作片的定义）
7、同步骤2
8-11、[19995]Prompt为4-6生成的图片描述、以及图片格式要求——（可能是生成了图片）
12、同步骤3（生成项目符号）
13、后面的步骤是对2-6的重复，直至完成整个PPT。


popai项目中关于function call的一些疑问
1、在哪里定义了tools或functions？
OpenAiApiFunctions
ai.bloom.app.openai.functioncall.XXXCall
2、在哪里将tool或function列表传递给大模型的？
同时支持tools和functionCall。在ApiSupplier中根据入参chatCompletionRequest填入具体方法。
3、如果大模型返回方法调用，在哪里识别的？
拼接：OpenAiClient#customMapStreamToAccumulator：将流消息中的functionCall或toolCall（可能有多个）拼接成一个完整的消息。
执行：ApiSupplier#handleOnlyFunctionCall：分别处理functionCall或toolCall（toolCall是一次性拿到多个方法调用的结果，functionCall需要考虑递归调用）
ApiSupplier#handleFunctionCall做了什么？如果是functionCall，区分PromptTemplate中的nextTemplateId是否为空的情况。
streamingChat(...)                             // 1. 获取一个流式 ChatCompletion Flux（Flowable）
    .startWith(Flowable.just(new ChatMessageAccumulator()))  // 2. 以 accumulator 开头
    .switchMap(acc -> 
        handleFunctionCall(acc, ...)       // 3. 进入函数调用处理逻辑（递归处理函数调用）
    );
.switchMap一点拿到第一个chunk，就调用handleFunctionCall去处理functioncall逻辑。
4、PromptTemplate中的FunctionCall和ext.processFlow分别是如何执行的？（整体流程是什么样子？）
functionCall的get仅在assembleChatCompletionEntityByPromptTemplate方法中出现，目的是将方法进行结构化。并赋值：
（1）FunctionExecutor：在service jar包中定义
（2）ChatFunctionDynamic：在service jar包中定义，方法定义
初始化：BaseChatEntity.initParams()、【其他Entity都有】AiCreationChatEntity.customInitParams()、WorkflowChatEntity.customInitParams()
赋值给ChatCompletionRequest(functions、functionCall)，然后调用OpenAiClient时作为入参
-------
processFlow的运转过程：(nextTemplate)
WanZhiChatImageChatEntity.init()、AiCreationChatEntity.customInitParams()[PPT]
获取到配置的nextTemplateId，重置chatCompletionEntity中的promptTemplateId。
5、为什么要在Prompt中设置nextTemplateId？而不是在代码中写流程？
猜测：可以看做同一个处理逻辑？后一个输入强依赖前一个输出（且是中间数据）？
6、WorkFlow具体是怎么执行的？
以生成图片的流程为例进行描述：600000
{
    "workflowConfig": {
        "id": "genQueryRewrite",
        "nodes": [
            {
                "id": "start",
                "type": "START",
                "displayRes": false,
                "ignoreError": false
            },
            {
                "id": "prepare",
                "type": "CODE",
                "description": "",
                "data": {
                    "code": "def main(user_prompt, product, images):\n    imagetext2video_prompt_template = \"\"\"\n    You are a prompt expert for text-to-video generation with {product}. Please generate a {product} text-to-video prompt based on the user's needs and the provided images：\n    {user_prompt}\n    Notes:\n    1. The prompt itself should be output in English.\n    2. The video does not require text by default. If the user explicitly requests text in the video, the language of the text in the video should be consistent with the language of the original text. \n    3. Use a simple and direct text prompt that describes the movement in the output.\n    4. Do not need to describe the contents of the image.\n    5. Only output the prompt without other texts.\n    \"\"\"\n\n    text2image_prompt_template = \"\"\"\n    You are a prompt expert for text-to-image generation with {product}. Please generate a {product} text-to-image prompt based on the user's requirements：\n    {user_prompt}\n    Notes:\n    1. The prompt itself should be output in English.\n    2. The image does not require text by default. If the user explicitly requests text on the image, the language of the text on the image should be consistent with the language of the original text.\n    3. Only output the prompt without other text.\n    \"\"\"\n\n    text = text2image_prompt_template if product == \"recraft\" else imagetext2video_prompt_template\n\n    text = text.format(user_prompt=user_prompt, product=product)\n\n    res = [\n        {\n            \"type\":\"text\",\n            \"text\":text\n        }\n    ]\n    if len(images) > 0 and images[0] != \"\":\n        res.append({\n            \"type\":\"image_url\",\n            \"image_url\":{\n                \"url\":images[0]\n            }\n        })\n\n    return json.dumps(res);\n",
                    "variables": [
                        {
                            "variable": "user_prompt",
                            "value_selector": [
                                "system",
                                "input"
                            ]
                        },
                        {
                            "variable": "product",
                            "value_selector": [
                                "system",
                                "product"
                            ]
                        },
                        {
                            "variable": "images",
                            "value_selector": [
                                "system",
                                "images"
                            ]
                        }
                    ]
                },
                "displayRes": true,
                "ignoreError": false
            },
            {
                "id": "rewriteLLM",
                "type": "LLM",
                "description": "",
                "data": {
                    "async": false,
                    "model": "gpt-4o-mini",
                    "maxTokens": 2000,
                    "stream": true,
                    "topP": 1,
                    "source": "OPENAI",
                    "userPrompt": "${prepare}",
                    "visionReq": true
                },
                "displayRes": true,
                "ignoreError": true
            },
            {
                "id": "rewrite",
                "type": "TEMPLATE",
                "description": "",
                "data": {
                    "template": "${rewriteLLM}"
                },
                "displayRes": true,
                "ignoreError": false
            }
        ],
        "edges": [
            {
                "id": "start-source-prepare-target",
                "source": "start",
                "sourceHandle": "source",
                "target": "prepare",
                "targetHandle": "target"
            },
            {
                "id": "prepare-source-rewriteLLM-target",
                "source": "prepare",
                "sourceHandle": "source",
                "target": "rewriteLLM",
                "targetHandle": "target"
            },
            {
                "id": "rewriteLLM-source-rewrite-target",
                "source": "rewriteLLM",
                "sourceHandle": "source",
                "target": "rewrite",
                "targetHandle": "target"
            }
        ]
    }
}

Start【工作流的起点】
  ↓
Prepare【生成 prompt 的 Python 代码逻辑】
  ↓
RewriteLLM【调用 GPT-4o-mini 模型生成最终文本】
  ↓
Rewrite【取 rewriteLLM 的输出作为最终模板展示】

在WorkflowflowChatEntity.startStreamingChat()中执行workflowRun，进而调用core jar包中的方法。
（1）prepare：用来拼接Prompt，可同时满足图像生成和视频生成两种场景。
（2）rewriteLLM：调用OPENAI
（3）rewrite：填充模版


7、Workflow机制（主要了解各个类型的节点如何运行）？
查看每个类型Node的run方法：BaseNode#run()
(1)CodeNode：利用 SharedInterpreter（Jep 或类似工具）在 Java 中运行 Python 脚本，并传参调用 main 函数。
(2)LLM：根据WorkflowContext上下文中的参数，构造VERTEXAI或OPENAI的请求参数，并进行调用，支持stream或非stream的返回结果（result是自定义参数，会把大模型的请求request赋值给processData）
(3)TEMPLATE：支持Jinja（Python 生态） 和 FreeMarker（Java 生态）两种模板引擎。
(4)IfElseNode：底层使用Easy Rules框架实现，定义facts和rules，然后fire。
(5)ParallelNode：将工作流中的多个子任务并行执行的“控制节点”。但它本身不执行实际的业务逻辑，只传递结构和控制信息。作用：①把多个子任务的信息包装在 `ParallelData` 中传递。②在工作流执行器中识别此节点为“并行执行”的起点 ③通常由调度器读取其 `ParallelData`，并同时触发多个子节点执行。
(6)ITERATION：实现了 对 IterationNode 返回的迭代任务的调度执行逻辑，并且区分了 并行 和 串行 两种执行模式，最终返回每个迭代项的执行结果组成的列表。
900002执行流程：
START
↓
CODE: 提取图片URL
↓
CODE: 构建图文检查Prompt
↓
PARALLEL: 并行调用
↓                           ↓
ITERATION: 迭代图文检查       CODE: 结果聚合与筛选
        ↓
        LLM: Gemini 图片评估

8、Workflow机制和functionCall机制在popai中是什么关系？兼容or排斥？
9、拍照解题流程？
Volcengine的图片翻译API
vertex的图片识别：识别图片中重要的对象、解答问题等
10、向量数据库应用在哪些场景中？PineconeVectorStore已废弃，现在用的是MilvusVectorStore
handle_file：
main：
service/page：


--------------------------------------------------------------------------------------------------------------------------------------------

关于talkinglime的监控指标分析：


关于ppt agent的新逻辑：

https://api.01ww.org/api/v1/test/debugLog?env=boe&logId=Root=1-688b2ad9-5b675f58471412d2745863c7&before=1209600
1、Your role is based on user input to select the best fit theme
2、You are a top-tier PPT creation expert, proficient in information gathering, content organization, and visual presentation. Your goal is to quickly create a high-quality, complete PPT based on the topic provided by the user.
提供theme的详细模版、tools
3、You are a top-tier PPT creation expert, proficient in information gathering, content organization, and visual presentation. Your goal is to quickly create a high-quality, complete PPT based on the topic provided by the user.
prompt的要求和步骤2类似，但区别在于：本次将tool调用的结果传给大模型（role=tool）
4、You are a top-tier PPT creation expert, proficient in information gathering, content organization, and visual presentation. Your goal is to quickly create a high-quality, complete PPT based on the topic provided by the user.
prompt的要求和步骤2类似，但在3的基础上，增加了对tool：make_new_ppt_slide 的调用结果（每生成一张slide执行一次该方法）

示例：channel_id='13c54e84-3221-4c4f-9eab-313fa55b7ef6'


选择theme ext.pptAgentIntention.sysMsg
提供tools function
将用户查询转化为一组精确有效的查询组合 
（和上面的一样？）
从file列表选择最贴合主题的一个
根据文字内容判断生图or搜图-8ge ext.pptImageAuto.sysPrompt
根据文字内容生成图片检索关键词-5个
前端开发助手（模板内容替换）-10个 ext.pptAgentMakePptSlide
图片打分-100个？
输出originalImgUrl-3个
调用生成每一页ppt的tool


生图 900010 ext
 900009 ext



目前使用了哪些大模型：popai-agent-claude
ppt agent：kimi-v2、gcp、aws、claude、gemini、
为什么会选择这些模型？

agent流程：
意图识别：model兜底使用gpt-4-mini，content、reasoning_content、tool_calls
图片：生图 or 搜图
systemMessage：promptTemplate
historyMessage：从messageForLing表中查询
userMessage：
tools：chatCompletionEntity.getFunctionCall()。首次对话时删除tools中的make_new、normal、comprehensive，其他对话轮次删除make_new
loop chat：循环调用大模型并处理可能得tool call。
    normal search：内部大模型服务
    Comprehensive search：file search、web search
    make ppt slide：
    insert / modify slide：都需要调用LLM
send event emit：NODE_START、MESSAGE、等


action： EVENT_TOOLS_DESC_MAP


上下文长度 = 输入 + 输出 的 token 总数。
max_tokens 一般是指期望模型返回的生成长度，不包括输入。
n 为每条输入消息生成多少个结果。
stop 为停止词，当全匹配这个（组）词后会停止输出，这个（组）词本身不会输出。



ppt中什么是media？
Subscriber中对WorkEventChunk的处理逻辑？

构造多个WorkEventChunk的顺序？
PptAgentWorkflowChatEntity.run()方法中：
1)nodeId:PptAgent,workflowEvent:NODE_START,toolGENERATE:,action:"✏️ Task Start..."
2)pptIntention:
nodeId:PptAgent,workflowEvent:PARALLEL_START,tool:PONG(临时用),action:null
如果content非空：nodeId:PptAgent,workflowEvent:MESSAGE,tool:chunk.getContent(),action:0
如果model是kimi：nodeId:PptAgent,workflowEvent:MESSAGE_THINK,tool:chunk.getContent(),action:0
如果chunk.getReasoningContent()非空：nodeId:PptAgent,workflowEvent:MESSAGE_THINK,tool:chunk.getReasoningContent(),action:0
如果chunk.getToolCalls()非空：对每个toolCall，且functionName是MAKE_NEW_PPT_SLIDE，nodeId:PptAgent,workflowEvent:TOOL_CALLS,tool:functionName,action:EVENT_TOOLS_DESC_MAP.getOrDefault(functionName, "")
如果cot非空（cot是在上述过程中append content内容获得）：nodeId:PptAgent,workflowEvent:MESSAGE_THINK,tool:cot,action:0
3)chat loop：
和pptIntention方法中调用pptAgentCallLlm生成的event逻辑相同。此外，还包括（processToolCalls）：
nodeId:NORMAL_WEB_SEARCH,workflowEvent:TOOL_CALLS,tool:NORMAL_WEB_SEARCH,action:"\uD83C\uDF10 Web Searching..."
nodeId:COMPREHENSIVE_WEB_SEARCH,workflowEvent:TOOL_CALLS,tool:COMPREHENSIVE_WEB_SEARCH,action:"\uD83C\uDF10 Comprehensive Web Searching..."
nodeId:PPT_AGENT_CHUNK,workflowEvent:MESSAGE,content:pptCardList,action:0



2)nodeId:PptAgent,workflowEvent:MESSAGE,tool:,action:
2)nodeId:,workflowEvent:,tool:,action:
2)nodeId:,workflowEvent:,tool:,action:
2)nodeId:,workflowEvent:,tool:,action:
2)nodeId:,workflowEvent:,tool:,action:













































