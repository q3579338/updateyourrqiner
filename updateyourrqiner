import paramiko
import xlwt
import xlrd
import logging
import os
import re
import json
import time
from time import strftime , localtime
from typing import Union



#读取的文件名字
READ_DATA_XLS = "data.xls"
#生成的文件名字
NAME = f"{strftime('%Y_%m_%d',localtime())}.xls"

WRITE_DATA_XLS = NAME

#在Liunx上创建的临时文件
START_FILE = "start.sh"

#设置日志格式与等级
logging.basicConfig(level=logging.INFO , format='%(asctime)s - %(levelname)s - %(message)s')
logger = logging.getLogger(__name__)

#生成新的excel文件
excel = xlwt.Workbook(encoding = 'utf-8')
tables = excel.add_sheet('data')
tables = excel.get_sheet('data')

# 表格各列的索引
IP_INDEX       = 0
PASSWORD_INDEX = 1
STATUS_INDEX   = 2
CHECK_INDEX    = 3

#生成的新excel表格的表头,可以根据自己的需求修改
tables.write(0 , IP_INDEX      , "IP")
tables.write(0 , PASSWORD_INDEX, "PASSWORD")
tables.write(0 , STATUS_INDEX  , "STATUS")

# 生成表格的记录索引
INDEX = 1

excel.save(WRITE_DATA_XLS)

#ssh 类
class Client:
    def __init__(self, ip : str, port : int = 22 ,password : str = "", username : str = 'root'):
        self.ip = ip
        self.port = port
        self.password = password
        self.username = username

    def login(self) -> bool:
        try:
            self.client = paramiko.SSHClient()
            self.client.set_missing_host_key_policy(paramiko.AutoAddPolicy())
            self.client.connect(self.ip , self.port , username = self.username , password = self.password , timeout = 60)
            logger.info(f"Login {self.ip} success")
            return True
        except Exception as e:
            logger.error(e)
            self.client = None
            return False
    
    def exec(self, cmd : str, **args):
        try:
            if self.client is None:
                return ""
            stdin, stdout, stderr = self.client.exec_command(command = cmd, **args)
            return stdout.read().decode('utf-8')
        except Exception as e:
            logger.error(e)
            if not self.login():
                return ""
            else:
                return self.exec(cmd, **args)

    def close(self):
        if self.client is None:
            return
        self.client.close()
    
    def __del__(self):
        if self.client:
            self.client.close()

# 通用函数类
def GetMaxNumber(content : str) -> int: 
    Max = -1
    pat = re.compile("btfs(\d+){0,}") 
    result = pat.findall(content)
    if len(result) == 0:
        Max = 0
    else:
        for i in result:
            if i == "":
                Max = 0
            elif int(i) > Max:
                Max = int(i)
    return Max

def GetPort(content : str) -> int:
    pat = re.compile('''"\/ip4\/0.0.0.0\/tcp\/(\d+)",''')
    result = pat.findall(content)
    if len(result) == 0:
        return 0
    else:
        return int(result[0])

if __name__ == '__main__':
    # 加载数据保存的文件 xls
    data = xlrd.open_workbook( READ_DATA_XLS, formatting_info=True)
    table = data.sheets()[0]                      #通过索引获取表
    nrows = table.nrows                           #获取行数
    ncols = table.ncols                           #获取列数

    # 读出全部所需数据
    IP_LIST = table.col_values(0)                 #IP列表
    PW_LIST = table.col_values(1)                 #密码列表

    if len(IP_LIST) != len(PW_LIST):
        logger.error("IP列表和密码列表长度不一致")
        exit(1)
    
    for i in range(nrows):
        IP = IP_LIST[i]
        PW = PW_LIST[i]

        if (IP == "" or not IP) or (PW == "" or not PW):
            # IP或密码为空 添加处理步骤 例如记录到excel
            tables.write(INDEX , IP_INDEX       , IP)
            tables.write(INDEX , PASSWORD_INDEX , PW)
            tables.write(INDEX , CHECK_INDEX , "IP或密码为空")
            excel.save(WRITE_DATA_XLS)
            INDEX += 1
            logger.error("IP或密码为空")
            continue

        if i > 0:
            print("\r")
        client = Client(ip = IP, port = 22 , password = PW , username = "root")
        Login = 3
        while Login > 0:
            if not client.login() and Login > 0:
                Login -= 1
                logger.error(f"登录失败，剩余重试次数：{Login}")
                continue
            else:
                break

        if Login == 0:
            # 登录失败 需要添加处理函数 例如记录到excel
            tables.write(INDEX , IP_INDEX       , IP)
            tables.write(INDEX , PASSWORD_INDEX , PW)
            tables.write(INDEX , CHECK_INDEX , "登录失败")
            excel.save(WRITE_DATA_XLS)
            INDEX += 1
            logger.error("登录失败")
            continue


        
        command = "screen -S qubic -X quit"
        result = client.exec(command)
        command = "wget IP:PORT/rqiner-x86 -O rqiner-x86"
        result = client.exec(command)
        command = "chmod +x rqiner-x86"
        result = client.exec(command)
        command = "screen -dmS qubic -L ./rqiner-x86 -t 2 -i ADDRESS"
        result = client.exec(command)
        time.sleep(1)
        #print(result)
        # 登录成功

        INDEX += 1
        excel.save(WRITE_DATA_XLS)
        client.close()
    print("\r")

