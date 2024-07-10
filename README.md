from selenium import webdriver
from selenium.webdriver.chrome.service import Service as ChromeService
from webdriver_manager.chrome import ChromeDriverManager
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from bs4 import BeautifulSoup
import pandas as pd
import time

# 初始化 Chrome WebDriver
options = webdriver.ChromeOptions()
options.add_argument('--headless')  # 无头模式运行
driver = webdriver.Chrome(service=ChromeService(ChromeDriverManager().install()), options=options)

# 设置 URL
url = 'https://iftp.chinamoney.com.cn/english/bdInfo/'

# 加载页面
driver.get(url)

try:
    # 等待 Bond Type 多选框出现
    bond_type_checkbox = WebDriverWait(driver, 10).until(
        EC.presence_of_element_located((By.XPATH, '//input[@id="bondTypeTreasuryBond"]'))
    )
    bond_type_checkbox.click()

    # 等待 Issue Year 多选框出现
    issue_year_checkbox = WebDriverWait(driver, 10).until(
        EC.presence_of_element_located((By.XPATH, '//input[@id="issueYear2023"]'))
    )
    issue_year_checkbox.click()

    # 点击搜索按钮
    search_button = WebDriverWait(driver, 10).until(
        EC.element_to_be_clickable((By.XPATH, '//button[contains(text(), "Search")]'))
    )
    search_button.click()

    # 等待表格加载完成
    table = WebDriverWait(driver, 10).until(
        EC.presence_of_element_located((By.CSS_SELECTOR, 'table.san-sheet-alternating'))
    )

    # 获取页面内容
    html_content = driver.page_source

except Exception as e:
    print(f"Error: {e}")
    driver.quit()
    exit()

# 使用 BeautifulSoup 解析页面内容
soup = BeautifulSoup(html_content, 'html.parser')

# 查找所有表格
tables = soup.find_all('table')
print(f"Found {len(tables)} tables.")

# 定位目标表格
target_table = None
for t in tables:
    headers = [header.text.strip() for header in t.find_all('th')]
    if headers == ['ISIN', 'Bond Code', 'Issuer', 'Bond Type', 'Issue Date', 'Latest Rating']:
        target_table = t
        break

# 提取数据
if target_table:
    rows_data = []
    rows = target_table.find_all('tr')
    for row in rows[1:]:  # 跳过表头
        cells = row.find_all(['td', 'th'])
        row_data = {
            'ISIN': cells[0].text.strip(),
            'Bond Code': cells[1].text.strip(),
            'Issuer': cells[2].text.strip(),
            'Bond Type': cells[3].text.strip(),
            'Issue Date': cells[4].text.strip(),
            'Latest Rating': cells[5].text.strip()
        }
        rows_data.append(row_data)

    # 将数据存入 DataFrame
    df = pd.DataFrame(rows_data)

    # 保存为 CSV 文件
    df.to_csv('treasury_bonds_2023.csv', index=False)
    print("Data has been saved to treasury_bonds_2023.csv")
else:
    print("No relevant table found")

# 关闭 WebDriver
driver.quit()
