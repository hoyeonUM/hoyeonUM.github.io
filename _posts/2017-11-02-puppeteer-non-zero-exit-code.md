---
layout: post
title: "puppeteer the Node.js process with a non-zero exit code"
author: "Chester"
---

Lazy loading 싸이트를 긁어오기위해 요즘 핫하다는 puppeteer 를 사용해보고자 설치했다.
내 개발환경은 현재 윈도우와 리눅스 혹은 vagrant 등 여러환경을 가지고있지만, 프로젝트가 윈도우에 설치되어있어서 윈도우로 개발시작

윈도우에 잘 설치한 이후 테스트를 끝낸이후 QA 서버 (centos) 환경에 배포완료.

`puppeteer the Node.js process with a non-zero exit code.` 와 같은 오류코드가 나온다.


`./node_modules/puppeteer/.local-chromium/linux-508693/chrome-linux/chrome -v --headless --no-sandbox --disable-setuid-sandbox` 실행명령어는 다음과 같다.

찾아본 결과 다음과 같이 처리. 

1. `yum install -y nss`


2. `yum install ipa-gothic-fonts xorg-x11-fonts-100dpi xorg-x11-fonts-75dpi xorg-x11-utils xorg-x11-fonts-cyrillic xorg-x11-fonts-Type1 xorg-x11-fonts-misc -y`


3. `yum install pango.x86_64 libXcomposite.x86_64 libXcursor.x86_64 libXdamage.x86_64 libXext.x86_64 libXi.x86_64 libXtst.x86_64 cups-libs.x86_64 libXScrnSaver.x86_64 libXrandr.x86_64 GConf2.x86_64 alsa-lib.x86_64 atk.x86_64 gtk3.x86_64 -y`


무슨 설치가 이렇게 많이빠져있는지는 모르겠지만 정상동작확인완료.


