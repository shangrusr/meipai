# meipai
just try
My first code.

#-*-coding:utf-8-*-
#-*-encoding=utf-8
import requests
import xlsxwriter as wx
import re
import time
from bs4 import BeautifulSoup

def meipai_search(keyword, page_total):
  global threads, mediaLinks
  for i in range(1, page_total+1, 1):
    url = "http://www.meipai.com/search/mv?q=" + keyword + "&page="+str(i)
  
    res = requests.get(url)
    res.encoding = "utf-8"
    soup = BeautifulSoup(res.text,"html.parser")
    soup = soup.select(".pr.no-select.loading.J_media_list_item")

    for thread in soup:
      try:        
          titles = thread.select(".content-l-p.pa")[0]['title']
          posters = thread.select(".content-name-a.js-convert-emoji")[0]['title']
          contentss = thread.select(".content-like.pa meta")[0]['content'][:-2]
          reviewss = thread.select(".conten-command.pa.js-span-a span")[0].text
          ID = thread.select(".content-like.pa")[0]['data-id']
          link = "www.meipai.com/media/" + ID

          mediaIds.append(ID);

          reviews =reviewss.replace('评论','0')
          contents =contentss.replace('喜欢','0')

          temp = [titles,posters,contents,reviews,link]

          threads.append(temp)
          # print(threads)
      except:
        continue

def meipai_comments(mediaIds):
  global items
  for mediaId in mediaIds:
    for page in range(1, 1000):
      reviewUrl = "http://www.meipai.com/medias/comments_timeline?page=" + str(page) + "&count=50&id=" + str(mediaId)
      commentUrl = "http://www.meipai.com/media/" + str(mediaId)
      response = requests.get(reviewUrl).json()
      print(str(len(response)),reviewUrl)
      if len(response) == 0:
        print('out')
        break
      for item in response:
        content = item['content_origin']
        userName = item['user']['screen_name']
        creattime = item['created_at']
        commentData = [commentUrl,userName,content,creattime]
        items.append(commentData)
        # print(items)

def getExcel(data, keyword):

  if (len(data[0]) == 4 ):
    lieming=['commentUrl','userName','content','creattime']
    filename = keyword+'_Comment_'+str(time.time())
  elif (len(data[0]) == 5 ):
    lieming=['titles','posters','contents','reviews','link']
    filename = keyword+'_Thread_'+str(time.time())

  try:
          workbook = wx.Workbook(filename+'.xlsx')
          worksheet = workbook.add_worksheet()
          # print(str(len(lieming)))
          for i in range(len(data)):
              for j in range(len(lieming)):
                  # print(data[i][j])
                  if i==0:
                      worksheet.write(i, j, lieming[j])
                  worksheet.write(i+1, j, data[i][j])
          workbook.close()
          print('excel Done')
  except Exception as err:
          print("excel "+err)


def main():
  global mediaIds,threads,items
  userid = []
  threads = []
  posts = []
  mediaIds = []
  items = []
  keyword = "网易+人物盛典"
  try:
      meipai_search(keyword, 41)
      meipai_comments(mediaIds)
      getExcel(threads, keyword)
      getExcel(items, keyword)
  except Exception as err:
    print('!!!!'+err)


if __name__ == '__main__':
    main()

