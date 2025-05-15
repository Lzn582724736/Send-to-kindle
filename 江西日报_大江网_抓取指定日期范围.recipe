from calibre.web.feeds.recipes import BasicNewsRecipe
#导入日期时间模块和正则表达式，用于判定链接是否为当日
import datetime,re

class jiangxidaily(BasicNewsRecipe):

    datetime_t = str(datetime.date.today()).split('-')  #对当天日期进行拆分，返回一个['2017', '10', '09']形式的列表，如果指定一个具体的日期进行抓取的话写为str(datetime.date(2017, 10, 07)).split('-')，或者直接写一个['2017', '10', '07']列表
    days_delta = 3 # 定义抓取区间，非calibre自带参数，在parse_index(self)中用于判断，具体见下
    #以下用于算出抓取新闻区间前后两个日期，在封面底端显示：抓取新闻日期区间\n2017-11-6至2017-11-13
    today = datetime.date(int(datetime_t[0]),int(datetime_t[1]),int(datetime_t[2]))
    before = datetime.date.today()-datetime.timedelta(days = days_delta)

    title = '江西日报'.decode('utf8') + '-'.join(datetime_t) + '前'.decode('utf8') + str(days_delta) + '天'.decode('utf8')
    url_prefix = 'http://www.jxnews.com.cn/jxrb/index.shtml' #url前缀
    description = '抓取江西日报（'.decode('utf8') + url_prefix + '）'.decode('utf8') + '-'.join(datetime_t) + '前'.decode('utf8') + str(days_delta) + '天的新闻'.decode('utf8') + str(before) + '至' + str(today)
    no_stylesheets = True #不采用页面样式表
    keep_only_tags = [{ 'class': 'p14' }] #保留的正文部分
    #移除上下多余元素，典型的web1.0产物
    remove_tags = [dict(name='td', attrs={'bgcolor':'eeeeee'}),dict(name='td', attrs={'height':'50'}),dict(name='td', attrs={'bgcolor':'f8f8f8'})]
    #remove_tags_after = {'class': 'ScrollDoor_Con'}
    #delay = 1
    #抓取出来的页面文章段落会居中，非常难看，即使在本句当中指定p元素的对齐方式也还是有部分文章依然居中，只能眉毛胡子一把抓，全部分散对齐算了这样做的结果是文章标题也没法居中了
    extra_css = 'p{ text-align: justify;}'
    #10线程下载
    simultaneous_downloads = 10
    remove_javascript = True
    max_articles_per_feed  = 999 #最大文章数，默认为100

    #压缩图片
    #compress_news_images= True
    #在首页所显示的日期格式，缺省格式为日，月，年：timefmt = '[%a, %d %b %Y]'，windows平台上此项不能包含中文字符，否则生成不了有日期的封面。linux下可以
    timefmt = '[%Y %b %d %a]'
    # 声明这个订阅列表的作者
    __author__ = 'suchao.personal@gmail.com'

    # 以下函数用于生成默认封面。关键的是img_data。
    def default_cover(self, cover_file):
        '''
        Create a generic cover for recipes that don't have a cover
        '''
        try:
            from calibre.ebooks.covers import create_cover
            title = '江西日报'.decode('utf8')
            date = '抓取新闻日期区间' + '\n' + str(self.before) + '至' + str(self.today)
            img_data = create_cover(title, [date])
            cover_file.write(img_data)
            cover_file.flush()
        except:
            self.log.exception('Failed to generate default cover')
            return False
        return True


    #下面的函数为recipe必要函数，返回的内容直接用于生成电子书
    def parse_index(self):
        #下面的for循环用soupfind找到各版面的url并生成列表，抛弃
        #以下，测试的时候几个地方漏写了self，因为暂时还没学，根据上面的class看，这个self应该也是面向对象编程的一环，提醒自己在一些变量和函数引用是不能丢，否则calibre报错
        soup = self.index_to_soup(self.url_prefix)
        banmiankuai = soup.find('table',{'style':'border:1px solid #ffffff;'})
        urlist = [] #各版面链接
        for link in banmiankuai.findAll('a'):
            if not 'jxrb' in link['href']:
                continue
            urlist.append(link['href'])

        #这个articles列表必须放在这个位置，放下下面的for循环里面会造成最终结果缺少东西，试了很多次的结果，原因待分析
        articles = []
        #下面的for循环用于给soup.find提供多个参数,即包含最终文章的链接网页框架
        for ur in urlist:
            soup = self.index_to_soup(ur)
            table = soup.find('table',{'cellspacing':'2','width':'96%'})#抓取的正文链接框架部分

            #以下for循环用于判定链接日期是否为当日

            for td in table.findAll('td',"p14"): #td,p14两个条件找出来的标签包含了链接块中的日期

                find_today = re.compile('(\d\d)-(\d\d)</td>')#构建找到末尾发布日期的正则表达式
                month = find_today.search(str(td))#把上面构建的表达式作用于findAll找出来的内容
                #try/except结构主要是用于正则表达式查找，如果不用这个结构，在部分标签当中查找不到内容的时候，下面引用查找结果group()就会报错，造成崩溃。
                try:
                    d1 = datetime.date.today()  # 获取今天的日期
                    d2 = datetime.date(int(self.datetime_t[0]), int(month.group(1)), int(month.group(2)))  # 获取新闻的日期
                    days_betwen = (d1 - d2).days #获取时间差，结果为整数
                    if days_betwen <= self.days_delta : #限定抓取几天内的新闻，当天的则为days_betwen == 0
                        soup2 = self.index_to_soup(str(td))

                        for link in soup2.findAll('a'):
                            if 'index' in link['href']:
                                continue
                            #contens[]是BeautifulSoup的一个属性，我理解为用于去除标签，两层标签就来两次contents[0]，详见https://www.crummy.com/software/BeautifulSoup/bs4/doc.zh/。strip() 是通用字符串方法，不加参数则用于去除头尾空格
                            til = link.contents[0].strip() + '(发布日期：' + str(d2) + ')'
                            url = link['href']
                            a = { 'title':til , 'url': url }

                            articles.append(a)
                except:
                    pass


        ans = [('江西日报', articles)]

        return ans
