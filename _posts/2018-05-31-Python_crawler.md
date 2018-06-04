---
layout: post
title: "Python_cralwer 실습"
date: 2018-05-31 00:00:00
img:
tags: [Python]
categories:
- Python
---

## Github 주소
https://github.com/kahee/fc-webtoon-crawler

## 실습 문제
```console
웹툰, 에피소드 이미지 클래스 작성, 에피소드 클래스 인스턴스를 내부에 가짐
class Webtoon
      attrs:
          webtoon_id
          title
          author
          description
          episode_list
      methods:
          update: 웹에서 가져온 데이터를 사용해 Episode인스턴스들을 생성, 자신의 episode_list에 추가
에피소드를 클래스로 구현

class Episode
    attrs:
        webtoon_id:     웹툰의 고유번호
        no:             에피소드의 고유번호
        url_thumbnail
        title
        rating
        created_date

    property:
        url (실제 에피소드 페이지의 URL을 리턴)
            파이썬 내장 urllib에 탑재되어있는 함수를 사용해서 생성
            ex) http://comic.naver.com/webtoon/detail.nhn?titleId=703845&no=18

위에서 dict형태로 만들던 로직을 클래스 인스턴스 생성방식으로 변경
episode_list리스트는 Episode인스턴스들을 자신의 요소로 가짐
>>> yumi = Webtoon(651673)
>>> yumi.title
유미와 세포들
>>> yumi.author
이동건
>>> yumi.update() <- update() 호출 안하고 yumi.episode_list에도 접근할 수 있도록 한다면?
>>> for episode in yumi.episode_list:
>>> print(episode.title)
306화 ...
305화 ...

# 숙제 1. extra1) 에피소드 이미지 클래스 추가
# class EpisodeImage (각 에피소드가 가진 이미지들 중 하나를 나타냄)
#       attrs:
#           episode
#           url
#
# Episode클래스에 image_list 속성 추가
#  상세페이지 크롤링 시 image_list를 EpisodeImage의 인스턴스로 채움

# 숙제 1. extra2) 웹툰 검색하기
# >>> webtoon = Webtoon.search_webtoon('대학')
# 1. 대학일기
# 2. 안녕, 대학생
#  선택: 1
# webtoon에 해당 웹툰의 id를 기반으로 생성자 실행 결과 (인스턴스) 할당

# 숙제 1. extra3) 에피소드의 이미지 다운로드 및 HTML생성 (매우오래걸림)
# >>> episode = Webtoon.episode_list[0]
# >>> episode.download()
# 특정 폴더에 해당 웹툰의 이미지들 다운로드하고 해당 이미지에 src속성을 가진 img태그들 목록을 갖는 HTML을 생성
```

----

## 코드
```python
class Episode:
    def __init__(self, webtoon_id, no, url_thumbnail,
                 title, rating, created_date):
        self.webtoon_id = webtoon_id
        self.no = no
        self.url_thumbnail = url_thumbnail
        self.title = title
        self.rating = rating
        self.created_date = created_date
        self.image_list = list()

    @property
    def url(self):
        """
        self.webtoon_id, self.no 요소를 사용하여
        실제 에피소드 페이지 URL을 리턴
        :return:
        """
        url = 'http://comic.naver.com/webtoon/detail.nhn?'
        params = {
            'titleId': self.webtoon_id,
            'no': self.no,
        }

        episode_url = url + parse.urlencode(params)
        return episode_url

    def get_image_url_list(self):

        file_path = f'./data/{self.webtoon_id}-{self.no}.html'

        if not os.path.exists(file_path):
            with open(file_path, 'wt') as f:
                response = requests.get(self.url)
                f.write(response.text)
            html = response.text
        else:
            html = open(file_path, 'rt').read()

        soup = BeautifulSoup(html, 'lxml')
        wt_viewer = soup.select_one('div.wt_viewer').select('img')

        for img in wt_viewer:
            new_ep_image = EpisodeImage(
                episode=self.no,
                url=img.get('src')
            )
            self.image_list.append(new_ep_image)
        return self.image_list

    def download(self):
        for num, img in enumerate(self.image_list):
            file_name = img.url.rsplit('/')[-1]
            dir_path = f'images/{self.webtoon_id}/{self.no}'
            os.makedirs(dir_path, exist_ok=True)
            headers = {'Referer': self.url}
            response = requests.get(img.url, headers=headers)
            file_path = f'{dir_path}/{file_name}'
            with open(file_path, 'wb') as f:  # wb: 쓰기
                f.write(response.content)  # 파일 저장


class Webtoon:
    def __init__(self, webtoon_id):
        self.webtoon_id = webtoon_id
        self._title = None
        self._author = None
        self._description = None
        self._episode_list = list()
        self._html = ''

    def _get_info(self, attr_name):
        if not getattr(self, attr_name):
            self.set_info()
        return getattr(self, attr_name)

    @property
    def episode_list(self):
        if not self._episode_list:
            self.crawl_episode_list()
        return self._episode_list

    @property
    def title(self):
        return self._get_info('_title')

    @property
    def author(self):
        return self._get_info('_author')

    @property
    def description(self):
        return self._get_info('_description')

    @property
    def html(self):
        if not self._html:
            # 인스턴스의 html속성값이 False(빈 문자열)일 경우
            # HTML파일을 저장하거나 불러올 경로
            file_path = 'data/_episode_list-{webtoon_id}.html'.format(webtoon_id=self.webtoon_id)
            # HTTP요청을 보낼 주소
            url_episode_list = 'http://comic.naver.com/webtoon/list.nhn'
            # HTTP요청시 전달할 GET Parameters
            params = {
                'titleId': self.webtoon_id,
            }
            # HTML파일이 로컬에 저장되어 있는지 검사
            if os.path.exists(file_path):
                # 저장되어 있다면, 해당 파일을 읽어서 html변수에 할당
                html = open(file_path, 'rt').read()
            else:
                # 저장되어 있지 않다면, requests를 사용해 HTTP GET요청
                response = requests.get(url_episode_list, params)
                # 요청 응답객체의 text속성값을 html변수에 할당
                html = response.text
                # 받은 텍스트 데이터를 HTML파일로 저장
                open(file_path, 'wt').write(html)
            self._html = html
        return self._html

    @classmethod
    def all_webtoon_crawler(cls, keyword):
        url = 'https://comic.naver.com/webtoon/weekday.nhn'
        response = requests.get(url)

        soup = BeautifulSoup(response.text, 'lxml')
        all_webtoon_list = soup.select('div.col_inner > ul > li > a')

        result = list()
        for webtoon in all_webtoon_list:
            title = webtoon.get_text()
            if keyword in title:
                href = webtoon.get('href', '')
                query_string = parse.urlsplit(href).query
                query_dict = dict(parse.parse_qsl(query_string))
                titleId = query_dict['titleId']
                check = [item for item in result if item['titleId'] == titleId]
                if not check:
                    result.append({
                        'titleId': titleId,
                        'title': title,
                    })
        return result

    @classmethod
    def search_webtoon(cls, keyword):
        """
        keyword와 일치하는 웹툰 제목 검색 함수
        :param keyword:
        :return:
        """
        search_result = cls.all_webtoon_crawler(keyword)
        result_list = list()
        if search_result:
            for webtoon in search_result:
                webtoon_title_id = cls(webtoon_id=webtoon['titleId'])
                result_list.append(webtoon_title_id)
        return result_list

    def set_info(self):
        """
        자신의 html속성을 파싱한 결과를 사용해
        자신의 title, author, description속성값을 할당
        :return: None
        """
        # BeautifulSoup클래스형 객체 생성 및 soup변수에 할당
        soup = BeautifulSoup(self.html, 'lxml')

        h2_title = soup.select_one('div.detail > h2')
        title = h2_title.contents[0].strip()
        author = h2_title.contents[1].get_text(strip=True)
        # div.detail > p (설명)
        description = soup.select_one('div.detail > p').get_text(strip=True)

        # 자신의 html데이터를 사용해서 (웹에서 받아오거나, 파일에서 읽어온 결과)
        # 자신의 속성들을 지정
        self._title = title
        self._author = author
        self._description = description

    def crawl_episode_list(self):
        """
        자기자신의 webtoon_id에 해당하는 HTML문서에서 Episode목록을 생성
        :return:
        """
        # BeautifulSoup클래스형 객체 생성 및 soup변수에 할당
        soup = BeautifulSoup(self.html, 'lxml')

        # 에피소드 목록을 담고 있는 table
        table = soup.select_one('table.viewList')
        # table내의 모든 tr요소 목록
        tr_list = table.select('tr')
        # list를 리턴하기 위해 선언
        # for문을 다 실행하면 episode_lists 에는 Episode 인스턴스가 들어가있음
        episode_list = list()
        # 첫 번째 tr은 thead의 tr이므로 제외, tr_list의 [1:]부터 순회
        for index, tr in enumerate(tr_list[1:]):
            # 에피소드에 해당하는 tr은 클래스가 없으므로,
            # 현재 순회중인 tr요소가 클래스 속성값을 가진다면 continue
            if tr.get('class'):
                continue

            # 현재 tr의 첫 번째 td요소의 하위 img태그의 'src'속성값
            url_thumbnail = tr.select_one('td:nth-of-type(1) img').get('src')
            # 현재 tr의 첫 번째 td요소의 자식   a태그의 'href'속성값
            from urllib import parse
            url_detail = tr.select_one('td:nth-of-type(1) > a').get('href')
            query_string = parse.urlsplit(url_detail).query
            query_dict = parse.parse_qs(query_string)
            # print(query_dict)
            no = query_dict['no'][0]

            # 현재 tr의 두 번째 td요소의 자식 a요소의 내용
            title = tr.select_one('td:nth-of-type(2) > a').get_text(strip=True)
            # 현재 tr의 세 번째 td요소의 하위 strong태그의 내용
            rating = tr.select_one('td:nth-of-type(3) strong').get_text(strip=True)
            # 현재 tr의 네 번째 td요소의 내용
            created_date = tr.select_one('td:nth-of-type(4)').get_text(strip=True)

            # 매 에피소드 정보를 Episode 인보스턴스로 생성
            # new_episode = Episode 인스턴스
            new_episode = Episode(
                webtoon_id=self.webtoon_id,
                no=no,
                url_thumbnail=url_thumbnail,
                title=title,
                rating=rating,
                created_date=created_date,
            )
            # episode_lists Episode 인스턴스들 추가
            episode_list.append(new_episode)
        self._episode_list = episode_list
```
- `getattr(object, name[, default])` : 주어진 이름의 object 어트리뷰트를 돌려줍니다. name 은 문자열이어야 합니다. 문자열이 객체의 어트리뷰트 중 하나의 이름이면, 결과는 그 어트리뷰트의 값입니다. 예를 들어, getattr(x, 'foobar') 는 x.foobar 와 동등합니다. 명명된 어트리뷰트가 없으면, default 가 제공되는 경우 그 값이 반환되고, 그렇지 않으면 AttributeError가 발생합니다.
- `os.makedirs(path[, mode])` : 인자로 전달된 디렉터리를 재귀적으로 생성합니다. 이미 디렉터리가 생성되어 있는 경우나 권한이 없어 생성할 수 없는 경우는 예외를 발생합니다.
- `urllib.parse.urlparse(urlstring, scheme='', allow_fragments=True)` :  URL에 있는 6개의 구성들을 파싱해준다. URL이 구조 **URL: scheme://netloc/path;parameters?query#fragment**
```python
>>> from urllib.parse import urlparse
>>> o = urlparse('http://www.cwi.nl:80/%7Eguido/Python.html')
>>> o
ParseResult(scheme='http', netloc='www.cwi.nl:80', path='/%7Eguido/Python.html',params='', query='', fragment='')
```
- `str. rsplit([sep[, maxsplit]])`: 오른쪽부터 문자열을 잘라서 반환
