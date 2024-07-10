import re
from datetime import datetime

def reg_search(text, regex_list):
    results = []
    for regex_dict in regex_list:
        result = {}
        for key, pattern in regex_dict.items():
            if key == '标的证券':
                # 匹配股票代码（如：600900.SH）
                match = re.search(r'股票代码：(\d{6}\.SH)', text)
                if match:
                    result[key] = match.group(1)
            elif key == '换股期限':
                # 匹配换股期限的日期范围（如：2023 年 6 月 2 日至2027 年 6 月 1 日）
                match = re.search(r'(\d{4})年(\d{1,2})月(\d{1,2})日至(\d{4})年(\d{1,2})月(\d{1,2})日', text)
                if match:
                    start_date = f"{match.group(1)}-{int(match.group(2)):02d}-{int(match.group(3)):02d}"
                    end_date = f"{match.group(4)}-{int(match.group(5)):02d}-{int(match.group(6)):02d}"
                    result[key] = [start_date, end_date]
        if result:
            results.append(result)
    return results

# 测试代码
text = '''标的证券：本期发行的证券为可交换为发行人所持中国长江电力股份有限公司股票（股票代码：600900.SH，股票简称：长江电力）的可交换公司债券。
换股期限：本期可交换公司债券换股期限自可交换公司债券发行结束之日满 12 个月后的第一个交易日起至可交换债券到期日止，即2023年6月2日至2027年6月1日止。'''

regex_list = [{'标的证券': '*自定义*', '换股期限': '*自定义*'}]

print(reg_search(text, regex_list))

#测试结果
[{'标的证券': '600900.SH', '换股期限': ['2023-06-02', '2027-06-01']}]

