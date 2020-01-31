---
title: Maven Error 정리
permalink: 
excerpt: {"maven", "error"}
last_modified_at: 
redirect_from:
  - /theme-setup/
toc: true
---



### The expression ${artifactId} is deprecated.
The expression ${artifactId} is deprecated. \\
Please use {project.artifactId} instead. \\
버전업이 되면서 작성법이 변경되었습니다. \\
build 등에서 설정시 {artifactId} 라고 적었던 것을 \\
{project.artifactId}라고 적어주기만 하면 됩니다. \\
version도 동일하게 발생하니, 변경해 주시면 됩니다. 
