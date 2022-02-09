# MELO
from selenium import webdriver
import urllib
import os, io
from selenium.webdriver.support.ui import Select
import time
from PIL import Image
import cv2
import ddddocr

option = webdriver.ChromeOptions()
#option.add_argument("--incognito")    <=這是無痕視窗
ua = "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.45 Safari/537.36"
option.add_argument("user-agent={}".format(ua))         #加入header

option.add_experimental_option('excludeSwitches', ['enable-automation'])
driver = webdriver.Chrome(options=option,executable_path = 'C:/Users/User/Desktop/ChromeDriver/chromedriver.exe')
#cookies={'name':'APID','value':'UP639f1615-ab9f-11e8-820e-029a9b0f783e'}
url = "https://irs.thsrc.com.tw/IMINT/"
driver.get(url)
#driver.add_cookie(cookie_dict=cookies)
#driver.get(url)

#加入偵測clickable, 針對Pop up的cookie同意詢問欄位
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
wait = WebDriverWait(driver, 10)
POPup = wait.until(EC.element_to_be_clickable(("xpath", '//*[@id="btn-confirm"]')))
POPup.click()
#driver.find_element_by_xpath('//*[@id="btn-confirm"]').click()   #Pop up的cookie同意詢問欄位


def Page_1():
    driver.find_element_by_xpath('//*[@id="content"]/tbody/tr[1]/td[2]/span/select').click()  #點選起程站
    select_from = Select(driver.find_element_by_name('selectStartStation'))    #下拉式選單
    select_from.select_by_visible_text(u"台北")

    driver.find_element_by_xpath('//*[@id="content"]/tbody/tr[1]/td[2]/select').click()  #點選到達站
    select_to = Select(driver.find_element_by_name('selectDestinationStation'))    #下拉式選單
    select_to.select_by_visible_text(u"台中")

    driver.find_element_by_xpath('//*[@id="returnCheckBox"]').click()  #點選去回程

    TO_date = driver.find_element_by_xpath('//*[@id="toTimeInputField"]')
    TO_date.clear()
    TO_date.send_keys('2021/12/20')    #填入出發日期
    driver.find_element_by_xpath('//*[@id="toTimeTable"]/select').click()  #點選出發時間
    TO_time = Select(driver.find_element_by_name('toTimeTable'))    #下拉式選單
    TO_time.select_by_visible_text(u"07:30")

    BACK_date = driver.find_element_by_xpath('//*[@id="backTimeInputField"]')
    BACK_date.clear()
    BACK_date.send_keys('2021/12/22')    #填入出發日期
    driver.find_element_by_xpath('//*[@id="backTimeTable"]/select').click()  #點選出發時間
    BACK_time = Select(driver.find_element_by_name('backTimeTable'))    #下拉式選單
    BACK_time.select_by_visible_text(u"16:30")


    img = driver.find_element_by_id('BookingS1Form_homeCaptcha_passCode').screenshot_as_png   #根據驗證碼id 截圖(screenshot_as_png)
    imageStream = io.BytesIO(img)                  #用io套件轉為二進制 暫存
    im = Image.open(imageStream)
    im.save("驗證碼.png")
    im.show()

    ocr = ddddocr.DdddOcr()
    with open('驗證碼.png', 'rb') as f:    #C:/Users/User/PycharmProjects/pythonProject3/
        img_bytes = f.read()
    VERIFY = ocr.classification(img_bytes)
    print(VERIFY)

    blank=driver.find_element_by_xpath('//*[@id="securityCode"]')
    blank.clear()
    blank.send_keys(VERIFY)

    driver.find_element_by_xpath('//*[@id="SubmitButton"]').click()


Page_1()
