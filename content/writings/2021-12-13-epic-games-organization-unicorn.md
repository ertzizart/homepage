+++
title = '当GitHub组织成员太多导致页面出现问题'
date = 2021-12-13T10:18:00+08:00
article_author = "Hao Wu [ Ertzizart ]"
preface = ""
preface_author = ""
license = "CC BY-NC-SA 4.0"
license_url = "https://creativecommons.org/licenses/by-nc-sa/4.0/"
novel = false
draft = false
+++

满怀激动的心情加入EpicGames的Github组织，想着这样出名的图标像个徽章挂在自己的Github主页有多亮眼。结果因为组织成员默认设定不可见，导致其没有第一时间出现，但想着去设置时怎么也找不到自己，直接通过url想要强行显示又会报错出现下图一样的独角兽页面：

![unicorn](/assets/pics/2021-12-13-epic-games-organization-unicorn/unicorn.png)

那是因为这个组织人数太多了。

![tnum](/assets/pics/2021-12-13-epic-games-organization-unicorn/tnum.png)

而Github组织成员列表最高只能显示五万人。

![fnum](/assets/pics/2021-12-13-epic-games-organization-unicorn/fnum.png)

想要让EpicGames的徽章挂在自己的Github主页就需要用到Github API以及一个Token，下面就是将自己的成员可见性变为公开的某个方法，记得用自己的Token和用户名替换掉 `ghp_xxx...xxx` 和 `username`。

```bash
curl -H "Accept: application/vnd.github.v3+json" \
-H "Authorization: token ghp_xxx...xxx" \
-H "Content-Length: 0" \
-X PUT \
https://api.github.com/orgs/EpicGames/public_members/username
```