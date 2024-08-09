# puppeteer 实现自动登录

```javascript
import puppeteer from 'puppeteer-core'
import Tesseract from 'tesseract.js';

const getCode = async (url) => {
  const res = await Tesseract.recognize(url, 'eng', {}); 
  const {
    data: { text },
  } = res; 
  return text.replaceAll(/[^\d\w]/g, '').replaceAll(/\s/g, '')
}

async function getValidCode(page) {
  const selector = '.formWrap .el-form .el-form-item:nth-child(4) img'
  const img = await page.$(selector)
  const src = await page.$eval(selector, (node) => node.getAttribute('src'))
  const code = await getCode(src)
  if (code.length !== 5) { 
    // 刷新code码
    await img.click()
    await page.waitForTimeout(300)
    return getValidCode(page)
  }
  return code
}

// 处理验证码
async function typeCode(page) { 
  const code = await getValidCode(page)
  const input = await page.$('.formWrap .el-form .el-form-item:nth-child(4) .el-input__inner');
  await input.click({ clickCount: 3 })
  await input.press('Backspace')
  await input.type(code)
}

async function login(page) { 
  // 验证码
  await typeCode(page)
  // 登录
  await page.click('.formWrap .btnWrap .el-button')
  await page.waitForTimeout(2000)
  if (page.url() === 'https://screen.aibee.cn/#/login') { 
    // 登录失败
    return login(page)
  }
}

(async () => {
  const browser = await puppeteer.launch({
    headless: false,
    executablePath: '/Applications/Google Chrome.app/Contents/MacOS/Google Chrome'
  })
  const page = await browser.newPage()
  await page.setViewport({ width: 0, height: 0 }); // 设置页面大小
  await page.goto('https://screen.aibee.cn', {
    waitUntil: 'networkidle0'
  })
  // 用户名
  await page.type('.formWrap .el-form .el-form-item:nth-child(2) .el-input__inner', 'admin@aibee.cn')
  // 密码
  await page.type('.formWrap .el-form .el-form-item:nth-child(3) .el-input__inner', 'gD2!cF0gip2T#p!s')
  // 登录
  await login(page)
  console.log("登录成功")
})()
```