```python
if __name__ == "__main__":
    import requests as req
    import json
    from lxml import etree
    from tqdm import tqdm
    import os
    import win32con
    import win32api
    import random
    path_list = os.listdir('.')
    if 'PearVideo DownLoad' in path_list:
        pass
    else:
        os.mkdir('./PearVideo DownLoad')
    try:
        while True:
            url = str(input("请输入要下载梨视频的链接："))
            video_id = url.split('_')[1]
            headers = {
                'Referer': f'https://www.pearvideo.com/video_{video_id}',
                'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/102.0.0.0 Safari/537.36'
            }
            with req.get(url=url, headers=headers) as resp_title:
                html = etree.HTML(resp_title.text)
                video_title = html.xpath('//*[@id="detailsbd"]/div[1]/div[2]/div/div[1]/h1/text()')
            video_s = f'https://www.pearvideo.com/videoStatus.jsp'
            video_params = {
                'contId': f"{video_id}"
            }
            with req.get(url=video_s, headers=headers, params=video_params) as resp:
                js = json.loads(resp.text)
                resp.close()
            systemtime = str(js['systemTime'])
            video_url = str(js['videoInfo']['videos']['srcUrl'])
            video_url = video_url.replace(systemtime, f'cont-{video_id}')
            color_list = ['BLACK', 'RED', 'GREEN', 'YELLOW', 'BLUE', 'MAGENTA', 'CYAN', 'WHITE']
            with req.get(url=video_url, headers=headers, stream=True) as response:
                video_size = int(response.headers["Content-Length"]) // (1024 * 1024)
                with open(f"./PearVideo DownLoad/{video_title[0]}.mp4", 'wb+') as f:
                    for data in tqdm(iterable=response.iter_content(1024 * 1024), total=video_size, unit='MB',desc=f'正在下载{video_title[0]}.mp4    文件大小{video_size}MB', leave=False, colour=color_list[random.randint(0, 7)]):
                        f.write(data)
                    f.close()
                response.close()
            print(f"{url}下载完成！")
    except:
        win32api.MessageBox(0, 'ERROR', 'ERROR', win32con.MB_YESNOCANCEL | win32con.MB_ICONSTOP)
```