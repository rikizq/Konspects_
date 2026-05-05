### Pytest.py
```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

chrome_options = Options()
BROWSER = webdriver.Chrome(options=chrome_options)
SITE_ADDRESS = 'https://practice-automation.com/'

def test_openMainPage():
    wait = WebDriverWait(BROWSER, 10)
    BROWSER.get(SITE_ADDRESS)
    assert BROWSER.current_url == SITE_ADDRESS

def getClickEventsButton():
    #Проверить существование кнопки
    pass

def clickButton(Button):
    Button.click()

open_main_page()
```

### Test_section.py
```Python
'''
1. Зайти на сайт practice_automation
2. Найти и кликнуть по кнопке Click Events
3. Проверить переход на страницу https://practice-automation.com/click-events/
4. Проверить все кнопки с животными на этой странице:
Cat - Meow!
Pig - Oink!
Dog - Woof!
Cow - Moo!
'''

import pytest
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

    
chrome_options = Options()
BROWSER = webdriver.Chrome(options=chrome_options)
SITE_ADDRESS = 'https://practice-automation.com/'
CLICK_EVENT_PAGE_ADRESS = 'https://practice-automation.com/click-events/'
CLICK_EVENTS_BUTTON_CSS = 'a.wp-block-button__link.wp-element-button[href="https://practice-automation.com/click-events/'


@pytest.fixture(scope='module')
def browser():
    chrome_options = Options()
    driver = webdriver.Chrome(options=chrome_options)
    yield driver
    driver.quit()

def test_openMainPage(browser):
    wait = WebDriverWait(browser, 10)
    browser.get(SITE_ADDRESS)
    assert browser.current_url == SITE_ADDRESS

def test_findAndClickEventsButton(browser):
    button = browser.find_element(By.CSS_SELECTOR, CLICK_EVENTS_BUTTON_CSS)
    browser.execute_script("argument[0].click;", button)
    wait = WebDriverWait(browser, 10)
    assert browser.current_url == CLICK_EVENT_PAGE_ADRESS

def test_animalButtons(browser):

    pass
```

