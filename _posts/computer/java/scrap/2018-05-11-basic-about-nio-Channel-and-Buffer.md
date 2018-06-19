---
layout: post
title:  "basic about nio Channel and Buffer"
categories: computer
tags:  computer java nio
author: "占小狼"
source: "https://www.jianshu.com/p/052035037297"
---

* content
{:toc}



> 简书 占小狼 转载请注明原创出处，谢谢！  

前言  

Java NIO 由以下几个核心部分组成： 

1 、Buffer

2、Channel 

3、Selector

传统的IO操作面向数据流，意味着每次从流中读一个或多个字节，直至完成，数据没有被缓存在任何地方。

NIO操作面向缓冲区，数据从Channel读取到Buffer缓冲区，随后在Buffer中处理数据。

本文着重介绍Channel和Buffer的概念以及在文件读写方面的应用和内部实现原理。

### Buffer

> A buffer is a linear, finite sequence of elements of a specific primitive type.

一块缓存区，内部使用字节数组存储数据，并维护几个特殊变量，实现数据的反复利用。 

1、**mark**：初始值为-1，用于备份当前的position; 

2、**position**：初始值为0，position表示当前可以写入或读取数据的位置，当写入或读取一个数据后，position向前移动到下一个位置； 

3、**limit**：写模式下，limit表示最多能往Buffer里写多少数据，等于capacity值；读模式下，limit表示最多可以读取多少数据。 

4、**capacity**：缓存数组大小
<!--more-->

![](data:image;base64,UklGRv4hAABXRUJQVlA4IPIhAADQtgCdASr6AUwBPp1Gnkulo6ahorWMENATiU3Z80L/jwJfk633/+nTaXGP+x2V/s37/3X/6MbnvzHfwDpfefwc3bP8jmJY7eB3JKj/ap8h5DkEvoo/y/qAf4DoW+Yr9h/WA/5/7ce7P+t9Mx/yf//7k/9w/6v//9wb+W/7b1p/VP/xHnj+oB///UA///Wv9Xv7J20/3Lw98Q/nn2/9cf+F8fvTf+59D/5N9sPxn+A/cb4y/qv+c/unjf8Iv6T1AvVv+f/r3ko/xfbtgB/N/6F/vf774xH796BfYX/Ue4B/Pv6x/tvVf/hf8jx5vwn+29gL+Z/27/jf4/8uPpj/nf+9/n/yo9t303/6f9d8Bv8+/sv/Q/xPtr+xz96fZC/c///l7N5AOWDi4cY+FvE/GNVCsIAnDDZNnCPj0NdO4VpPdJXbc4lChB2+rKOpPAV0zH5DL57Ud6vfBJ7fRC8KlxjAIqZRdm3CPDm++gqgTaandItPhYEWOuZMNQafPx6t0fz1SKkTeTB2mDwn8l6kHYCBNTfHr7kdwDNvT8ggAr7h3l/E0fA249P/HYlq8drA35+UNZaE50VH0lghs8/e3cNLtIjt3gden2pv+qRsCPFnYtkqSg45rVmevkeQu1qlH/hZVALjfhPBj/AK8LEXilf09eTgJNY7tVHs/BAmqj2fgbxNNDywB8xOrgW1OIQfw9Phj0mV4QIcVUEx8/ASiLyGRh12HomvJQ1xs6dxaTD2G97fAVfz2yyq5I9+m+zM/wCieaPH30gWWRji7L2B5XsCdc9emXshe8H3kesXsUsQ26Xr13Eh/t66bWSrzItb7DbwBt+Pt3yEUJDxSXyIVb79DOinVI5I3XeESTBLMc568TAKXrnd9+/DdxdQ/vSsz9yggFVqT9AmNX99A7nSURGqD2qMVgQz5y6YaMAC0LhXHy+kumXDICElDqhcjqR/6tvBk0Qi6T9KtQmcW7/zMUIiN29xdWoc+QBpzo0MvjT6mCUo2SxNLqwt3TluU3hpp/IH1GZurd/40+rbwRpl2Grk9k7RrMit2pkv6f9ncMPR59W22prdJ/nVx86Duout9y6Vi9izgnRSC9g6BS9QAH7+jVZS/R1foD3736eRQ3dovWzNQdYApxwZpkVFuAn+F+EoniCwXpCjxFoiPeBdZWyw1Abmhyxdww0gmOCu0xHQ3vcmNEUhxFGkF1mEIek5BVTlahHnEjnnOIUr4Zm82UQOyGgsIrEEB+ISrPZ3K2FNbwm/8Y4O3TUTS3SgzxhnqRq4KdcBPAlNEbgaWhkY5OFX3cLbUzjsKgcoMk3EeVYC4TL4/0nVHDACDuPhwPO/g3UKwzOcu/qYzaK8dstgz8g/7QmXVp9DiwGFLT59J/dioOKMuGIK7LV8ZnrUVSsY1knbvpPbLu0Ix2QOixXpWb2sB4dvwG0v/sA68kelix8qkY/0wIVx6BqZANDGiQFb0X6RSsXsWdVLww8DnwA2sUni8N1bq1xCl0q5olk4Z1XVLlAPJLdpDvSs3tPv6hHl/pVTdu9cDas3yUNd0P8QLH2xjIw+euBtWb5KGub51shhsmS+VKW30Ve7iTaad4aZy6ZES5kB+iGcIGJYtzQlJaqcyg5xMQQKZc31Sf0i/SMMLzcahsAGlpvw7rQzS8c5TYcZmrxrx66VS2n9uot2hR9jrJFs3LQcQBZcliZ39WC7gCjnXklfTbS4qqSfKW0AF+eqNa0CqfABKV20ZLBxo06sgseaiZRUV2hPeFVlwjiBB2D4+eoDV6tk5oVTqB3zF3eHAGCL7BX/pysXAtB+Qi5+va5A0yRUCedjq8E40cieVduMIOsf9vkYXfg0bYWm0TA1kzEVu5+rQDbmWXU2ZL7Q15aYVMMKk9Gvm+wcW4CtegfStYybvQoc2Um9mCEbujl607zx3XTA4naFpQ/Z7ua3hjrWXuQIcaBK9lQ8km4AAP76XgXPXeAbn6VTgpTl4cuArO1OpOPE3rrVKdrK/vAhEoIBRYp9L4KhHecXOpwK4PqYV8SakWLUn9lUkhZj66O8e5njHh2wuJxMq4P+EigavN4Eub6HpXntI0v489pzd8z0c3Lhjf9d0HYdFLO/vCgQrEv+SSB25VmV0PjFf/UKzF8hWLomQTWG+FcRuOM/qomlakQfkMontHduPXs/2/t+807E9l92hIjRksk3T0HdeHpw/AB7abgf2dZ69pDu3Db9eiw4n4kvVMdO8/sFgHQCnZKlwQkRxMIKXtjVrduF/wgy0fgiLfwoZValS7Jg3wq+a2wANQ6AHJBL8Ti815S2UrlqF74NDjmsuzNADyyvwO7xRKccpmDrp8uzAhbPnwCKTkkh/gNGn4FdfakwXnNSfeZF6OrNdq3jQU8A19h8oJjHjPHOOyBqwU+iA/5HcImPWwNEdQhc4AMN8bfgt47OeVvFv0DdM7XbZA1yWV/swGRraMOKsw/bTeKxzmpbZEGsL8AXkdWlVuMZ4o47XLrrQT1cHc8YPgYg/UXTCYprEiMYH1D0VMCqbEGXRT6y4n0p30V9XcJUj2hRD95PnMkIzVaSbADZnZ/RFBYm/us/tEbduqfCNNx4bI4IAlxNjRnwCYW4szzku67ySSj8Aj294K6gVj9pkvqE2h5GZoSl/XI5mpmumfF8pvFQDGSVVY2pbMzBuxWSeoxzH6pgeSerEQ2RTOqDDRXnoOW954+N1BFpPFc/509ZppSzqELeCxrrGBGYfXf7OO8/sS+3UTNDQ/cTnmQMrGucknaUPi574kGSfrcFMO6jH0TpkwXXMonPRNpuntpn84qy6/ETtkzhoxSDB1xIZTE65tgDzJ2zPm+xDVNdqkF+C3yO5Qict58eG+jFMeDCemmvTLX8F34E2ay436P8zw+pUg6abY1aAJr/Ihik3bt5GoT2PI2ZR6i6aL6A6mh7IwZmKP6LTcTe/b2N22qrVwnhbWF1PGq5vohG9a0p/n9rhQ1+k7acBJ6hNR50nFzHhCSbqTBOZluNGojK0TWsA0Z7uK7zC+f0lKHKOSwSNagArusR9phjMjIYykTeKUQDDXQ/rEBtb+2UxKG8OIGVhEPb9/Z9KsStqqnCO5LFFfjfbUv+VdPYTzXnEcI6uUVNzisH9iMUuAY4jNkbjRasa2Ug3X5IlvgiM8Oet/qlYo4Dvn03Dys4QQIDyHmyPE4Rv4La3ml2IO2Soxy7RK4w8G8kEyxqu3WdvhP7ybHae7djZYut2qC0OHUBMjRlsI1qMgKR+2W2n+yTLM0bW5DLnhHZE1WE5x6QpR7zRh996Sy4I0nlNYJeVVYS/RWx3LdFMkDTHE3wnQmfYKf8CH75tlpkl+kTfUMzz3+55/vNS/djatUMHjL8pdXOxfBtD3DvcHGLbtPG3YH14l07mGNiVvJGDFDl0RbwPqGWcUgI2xazTAptAvXyKLy0lqb8ajf4qbDn0A9zqplB0bGhtY55eh0RMR76jrpiSRn7328YX3VaExA4ShW9bIEIPRPgwG+/xDhSZmBFfpP0G45N0QGIM/1oCeannINcrgFMVxfLBs/VmKTPiDod1FMlS529zdfQrg6roDsP5zTSSz6UwJfJPViIbJTW1dWGGcXayu7b2RhYUmhJO0ud58CNXdKEuq8KdpzYET89iYO/5c9Xj3EtwFM+W0wv4Fq17AUHrgWt6H1gMJsHqst9StXALv6o5+NbsAo3mvk21sIZ5Bf6mlX/QyfpYX1X5+DFXvkjUaj19jfvKsDfkAIYKS2CsTRhBEWaVZq9b1byiLeen6N6qKGl32Pj52o2m3zBtLxfA5SBV8pN3+R+McrgfuS2vyv1tmubtnlPGG5yyl7Vye+jBiKd/3E+UzMUxiFBbW5wkV1JXT9qLboUeGWCY8O7xX8Z4W38eSdDmHR1vhdamBi5a3Rrxni/FWyofPnkp9+3Dl/tmCroS7ZfCDneSQ7gA0NmazdoofR1J4BmsDJy2JAlFMpNezJ+F5hxztB/XFJ2w3SdlEbQSac1yZMlUyzubFS+pBbAZjnIsGjO1Sl2D0S2Le2D78T3VbOfL6KN//H3jt8DBD4v7dUkcJJiO0v9huRNtcMz/fiU18w/kjBpdswC8FWB62Ok1zpeSSH+A0afgV19qUHbmGCJnc7qOWzQjxMQGL9Il/Pk0lFIZ7C6ATN4QXtmwfq/OhzT/BlV3KpiPrBsTxREEO6ORg6XF2kK8v2umO0TDLcx2AlWnAsRmzCwxF4jsD4Zz4c9DaTVewRjQLrEuH9T5w0LJKJDEd/LeocTdX100016zGsrPAvUczM8AiG2CqLQ6LaerAI0qLeYs1vhVOdZlI/pvjsHvp0P39Vj6HZ5/XIY9AucYB7gZQ8lc+2/WxJdq58Jc34GebLHczz9wuybucqibkKw6BBQz60p0LQCIEvd1OV3J7cy0s9DLcUGbhApqA+yayXC4qsqc1KztYQNRwyYyoG8j0rL2B+n+dT4f5nwUf4uVlYN0UXTACzH8oq8QBesy40FQEocEh+QaFDn55L7GpTd/QMPBHyCKRlsgfxt3uFcT+SIsnq+xWWh5FMKdsydBQe9xMT2SIBwV2J5KNAdKzWcJUVs0G5p1vOzNwWEeBzCTTc/z9g0A2GY/DGijkfJxmEJ3ulG8+b0ir4Gliq4H+p6JUF28x3XwGI+v/9serop4ZccTROEOWBZjAGOqp7UpuT7p8vU9qOedLEOcXxncspYOBcJNxGiBSm7IcHeffW0cPIQOdp0ar+2jIA3SLJi6DY8cb+uS58ORvT5A4ym6gXVNl62nQEHKyiQKuE5Y656Cx56CH46crlKnqJnZKn5UzuLFkKczHasH2NbPytvUZXnqm/juEAi96MJendVnr0dySP127IuT8W6v3w1/OuI0teoVVopTa0G5Ybt+tsS3fKVGDmemxxxnuE2CjFHyk+V03EjTq3n75O1BBJ5fRMGFEYB19N/rDaQPWUHFF4t73KttUxeygN+VrTdK+cGf+xh+1OCosRmWT+332DYMJdwYpgrHoFxDebLqgDDjLl2OhAcwMskosqNxuDHMQHVkzxk5GjjJu+UazJSY4SFjpJ8ApexdRbYegAAAABN+zqzjGJLFQLdkZHulUEJIGYVk9lIDGtIwHJSxKLRN/aFKYI3mKoreTkZwMkud/SyihChZIAAVO3RZMudwUEJEOcan9yyMVE+DlZIB8Q5LFOmt/58BnFzjF2lV2MQm26KyAAAAAAPya8RabKl/KEb8lLeDDLQnrOSAUCFGg/2kXxFdy/AIkyLrftJ6/7x7gN2tJjJXww96uZthS4/GMrAvohJcYXtbeApZPf1A7Q6urPP4Ywqj/8FbX/CuK1uaZijTc2c60wpHAMnuufyx5Xie4pXyYvzTrZOL0eY+99DxWa7kru3v86xGht3opp5VisHlOPlLzUm24ydkjCbww/i2BIgtHaAFi3eXQHvuSEbOZG+DIPN/+YmKnGofNLRyglLcSgBEuImLv03vtI3Lzr87MWj3NGWm0RNr3uruDTpxc1TE5ZV3ZplOG/O9TaE9UNitW3oIHrE8GSbb9sYIcCkJlgO7peheU2j7vAafW7eMtGfzKHfW1WZ5+fYZD/i+V8gBvMvBFuyctNvQtnrGz3X92Cexv/pZ0R+r2s/1Y6U0NAujXolF5oPW+ijx8iltuScEch8+CtH0N5HFevUORgCVp1gB0Txv/YEvQd5o1Qt2ULrN8uCAIP4mafHAndnmLOyasjOdvsNqMUEUow0UPvgDRNvhz/1PBXun2lyJqTokHq0W0BrYehDthT9kaCCZEfyL6Ba7jxjRrm9ce7TvOVJxDWfhjqsxYUvlvNDvvBtve2dtFpGm9gBDXAun+GsrxqdhsMk7JTeturdGQAtEH+F7XB0iklt5JUO5DRlSIybHpSeaC6QAKTs2SkygsuyKYeZL7qHkqSMBbz+Kv6HJUX7l0/A6Rn1kSBt0c7jQy6Ci6/mqVQgg9AuXlsHqDeBNb60NCZ3KNFW+ypwgZ9siz4/SMfW/H7k2COWveJHtVYf+4tPFw/Z/xyXUeA3RkMkJzxaIH9ItYsogQmRk/uHANSg5ih1zb1unKzCZ+zOYGECMJ8usKVIDP0xS3n6YsyFf20lEnvAXdU7nIEwhTd31PQPctQa+5ENZ6XWeNOO1uGkHodTyEwJXUaj8hLG3QMRcHOLQ0U0MOThLappa9j1a7VilyL+0hCx0GhvUJFhaVMtmirGj+qkUODkxq32ts/EKa4Ep0dXk2QNhkq3lE8yp8XFdBkbbDR3W6u/x8eHauGgK5R+scY15OXUIrahs1GXOE/cipavkE3PsEEyozaTzrMuOsE3mNU/WS0nTSugEydebdjI1qlpDkGsgiFWZ1WmWv4ayYT70UYSGgg2lS/+2wrkd1mVRiH4mAOOKqaNFZX1H90yqRoQCSMdkW4eqMgZbsj99/boc0jUEIjQT4V7vvrTAAsmKv1T2c8TG3nUYgcTNYN/HEvzYWWoTyC0FbJi/+mPb7jeoRXKn3blpYulzlU773UPyA2kuuitFIepJ83CNRABd/A6dRb6fH8vf6WMfa50/gZLjj9muzZcqZTOsviGa9kWjMqHq4v/vwrF/NyMrBYfjmHkr0w7owAAACnlBxRsNrriMwP8MspDdbxFTOubztxcF4kY/mvsXfUSdiNVzT6KC3FcawpwK1H9JmrFo4YUmwrD/fuSHp0pWMd+p/meVKFd4M4RAkuCSUesDgC06pkn0b3azdHBZ5qAq91u7/qIhdSXm/xwQVf3cAAn1WjHmH3l+eB+8Pm1q9xzR8DAKeDtUwLotAN+iUMHSTF+bwJmZiz9cmz8x+eu3KlZy7aaiQtm234q3SiZOQlGgM4q3FJgcGa02pR7UtsAN0HlVQnXQQH/mtTYF+ixM1zvloIAAnboFO7IjIpfe8i3gfZxfJR7TRUb6ofeMMhXJ6xKJRYPPPTjqaa4NFKmsZyDgbUjBIhnD+39A+np0ARZSi+1mLnogb46FHRCXpS46MGf6w1ifoN5fh3dQiSB2VauyRIkuW+IMHjEC4/U57EEAyb9JLhUar8vyiQ+NLWlrB4ulCE/CJA/cnxxZ6ZWjEY5YOTsG+gTkvUt5vw4eX3URRI1kL9sF/nbsTOFVT95IME9Uw3C8qIrQNt69BUuWi0M7YyaHVougpXnXszqSBXV6qFovoUBnglN9WyMkr7BVRoceY55V3ub4XRrfwkRyiOqtANFFdtJ9o7LzHBGctRjDZFr0LSfpKSv2oOCVKPqXa/ngBv/B77TaZ3W9GDIgnGr4uMV29cIUkQtLDhmWTRJPtAXUmFOC0B9PwQCf34JizBjCQYGoWyzB2foJ82ul6Fp4bgj11niXKEq3Ip2Zkf7k34WrHkORi3VKkMDEG+CpA5y7Gg6zZeSWqgujDT4rAGJ+Eq8n7wH7ZTGytvlZprDO9/Ol3xE7xkaS2ZEB7scOBsVV83ISTgyk7XSw81u5YshnTJzOFF99gNS5u3EUNxd8ejbSMKTd+fjQ4wj71wElsC1+RbB9ot4kHcXfgDoq5H68WhQFctOjaZfkhGQtaWCU8M+3iXW68nqylCxsAjCGwjcS6bDofshohifmn5oY8lxDg/4/JjXNgVgkrhjeNlUvw/gNQM+Ru1/LeYdDodfXzE9ndDuAOTbYjncn/jJzPSpJgP1zZP52YuLNckOIhnJtngBKa37OjGwifKv9DtoP76fGRKQRD3lvXJkqoaWCWggww6ZaASwUlpDIJJd/DuQA2vtYLNtfba4DYJ3lZP5jga48qtCO2TlLb9/UC5q95syEpzyDVgJ4uSmMYtg8mjWQ4VG+5as3T6feJ9QnGyCRWxrOPY5H7PUWw/Rem0RHyeg3BuApNQMzcKkvdXzxYVh/zDSH05GZdfBGtcTJC5Iemf2xC3GH6Im6Ggfmhc7PvBVYirq0xvbu9ZIoPPREO6noA55YvousrK0XdvTcjFni+bJBSD8mtFkPEYcmuvT85sdJlmL8JmszgHC8xhtJJpN6XZQvHC2rOGi3gvl4leACUH09wOr//+B4CXL83wnHHhZARVcJFuTVHjxKBWbzQy3cxs6iPAINtufkPY6KjQ0WeAT19UrtBLJWP2NJj3cAJGmODAgifqs4zT76XTSykIYnysMGF2Fn6bxtAo2JlHR33BvUECcugsDWnBFlU/3uH12KMmuNXrL0thJBdHRkubNSurPLIunB7lEwnguuplsTLYo84WZXhB4xl3Ca3lQdrlEPPV8Xs/VDHk5C2qYUGxqQaKbkRWi7YokAwSEbxJ2e4uu2SRvOgdmgjhDEmsbhfXtfCPgQPWe45Co3R4/BgA0In23vzRpkivA7c5u+AhrWarZqBk86EdfQNsuWsukXklvGl3JJ0zjMRvJYRtaDmC2BkZfN/zabG2FqAAqPy44jc85PuJtJVKVnt1b6WHxiWxbd2T1+M40Z7K5/VBtRY6Jzl3xAU+OpvO0U+uzYaTOWWhzBbiK35PhyBk94l7yeRUZzbTEmAhUr001qyCkQBYtZZWN5WtvzP5vNKYupt73xHHQBa1MVaJ51Y8bugHohrM+Jt1z4A9A8pyRrCOLqbHo8SSK0M/2c+THM+j0X2WU7MlvkuVGc/PSyhW7qxkq2VGY0egcv8NE+hWmpqpwcxEIrFwQJ/XI/8tHGJsEpB87tNUyvl6ap1/YCjkn45/MeDuWF0qElJKVpJE9ofgf3WseifsuaXRLUPlv+F5MyxMJJU2YxYFN6cBdwUj4cdDvWsO1XYKCcMfTzsz+p5jNVx14olaKk7JcSd1VX2vQtjWewRTrG6NqdPitjGcQaHCPd+EMc2xP9yEX641s4NIMTIrs12VuUSOw4YgYtSxSXA3caF6EpF2uvOx6wt4BKGSFRhgT72YzO3Kl0oAHmVV0ZzsYuJwaUrq9p6SMifZWeoDF6D4RpB3nf4h4DNbaDLABQvGuRWabY1V/DkzWkvF4N4o89hENAAAA/v+zSBzhT/R+Cd4Dpx+Ie456S5WVHoea81FE8Qpt7JRmzZcr1Gv7/dvGvLVZE+bboxqaB7GwuAqepY78sAsn8QWGQ+nZl8EcvsWqxwzgGC8dCFhFZZ6bKkw+kQR08mfBADmNiCzCpAaa47MSY7wwoAfDaZlnunlUMLwZ839aoPt+jvrIURScbz3J7GjfV6FdufQBUmuEpJCCPDDxDxDxDxDxDxDxDwntYMDB6pl3VKpKj87bjgCy9TG6jXnBpYrAAYlGWWijXrM8clTiJjI+B0FNQtboORInI7tf73ziFL4r2it9t+AA3V/SxcuRuh8ILmGbIYMLwl7JF/f8pESrly3EGXsVJQcPD+tAJ0EIXAca1WUK3yfvarCqmoiaW8eu5jriScGu2D9iFSbbHk/3EbgCJwieIwlmLL0ngJ80NAZ/qPmLj4NzRrksS4BwdFKgfdwgD2WAYc9I3OkQEVqd51Nl7R2RcCsgYgAmcKf4bAGkhpXIyAWiFtywRqZVjl4ebUcwHMNM+OIQTiBoXxsPhFrtiTU2ZxIAbR+87LJy1q/C0spmmGF3kxu8lexV/FRqV4yvQMdYgem8MOKa4mrChRpXLJcsz7Qyat/RHU0Allg2Fig9k5JTQaE5hPPlrR7kC/uFTlUy5+50di2WTER0Oy5/6aAH6swrO+LOEIE05bZnwO5DOJzy+mN6lOIEj5vmNzjppTvwL3TAmuhh+IY4N1Z0Z3Sq07aFtq9h3PyltH9ULYIrw2bQPo7a9+vJRtyXeJFFNHcAKt2m+hyi8qC2woLPXSbF0WHbcTfDKpdPpiShFzLDpttpBG5trk9WKw7xtmDYUXRERt//Vl1UWuw9bQ5j3RB1mEZziGE6cl/niZE06EShSarYnJwegWf/P4NJ9H3YFZspi4K89gSC92jFc8NXyX/PDBDRXmmzT8FXj1qzMX+CpAHwgLWsl7vh8HjBOi0hlNXtWxm6bnIq5L/HG1g8/7FN2PZF5Fm4VnMLwvTalt677yi4WmHMrqxtmNQwVthxNpfi9iRGZJcTgOnY03LLA1WzJHK9VcfFW7m4G0M8RHkPwBAPCwWzUDcXh6FyYwEwuEo8T9UoPJvIvdscb0d1JIa/O6Z57c1FkOIBczMGiUKsNGSMKjLI1evyqWcA7mR/pENDIrNum4hiqL8YCEqqnoAL8eILtf3iMopC1t2PfeJPFcjjAqUYW+yhLYB6MraVOIcZsxmwIiFgguXQlXilK4YMFJo6KsRjUsavZGfumYZhUxGHnx0SkraBQb/sQw6yRZ89ubYk6YKMEIIZkDJJW6bR3OM/kIXZZBrwaWNcyAO0eAoWX2wbz06k+KsvCBXmCET7V7LekPbgI+YH3TWstbtVpiBx69ETwhynlAcP+f74+D6BwoG3N8LB4MPc0atabbAt8QydFPDdI0k4pJHaJ2wWzUCHrjfz0yLdHntLPX10maVFL0Rjo4te99QzGjSWFTiwWzIm5VzR1syMg2uF+ivnnZJT2nQO4/XC04ubV4exwca4veW/HmwIPWIY7xMzuw3SasO+saQczSPUZIBqOjL6lOmiCgQtfivkzjpcVBA4eLRNr0xBOy0OgzvALErSzx2dxEM7GfOS2FjMP6eFqq8p9hFF+ikZwPdJ1YT1pjW50lWF+u3cDrGRDm6kbjzvUL7wGaalkEGbxrYbXeV4XjCGeDjn+rUAiBHjzt6sgxqZtborE9bjzXnHBLafP4UMejRcynxAlHmNgAEv44UF0gDFPMU859LzSxJmLE2hHXrFZApVrMMcUuBnghoiCItkLqEFoYolULIeMIpDbdAMS4E3LGNhVKcgnjKqeVQ9e6ChOjOH/2hxQBsU9jdasbNGfDPeOg8D0ga9DVLlumCLSFanWjvLZ+JPUPoeudOYRINeIgX999VNxYB8N33XoMWwtQoSuJmyu8msCn7vqVcr27L7o10emrmNQCbshXypFWiCox4cwX97q8TGB+6I5kAgxrn2jwCGyBBWZ2m/K261XoVC4yv2vw2Yw8t1M1dm0xfgomK/gxGq4bhli3s2iZsFW+YTeplOERnz5XT5eRskq3evFj1Ml4Wk770P6wkTsfyBjOE1cNNa2Fa0CbH7zdLBsibqlNEbMPwmg3qbaYYR2BVXE9U96LPgnnYsgMWpiQqP7UzynGJkddgswKbCdJjUQsWWzYy+emCEwJq1RsTIcMGyehzpUrCNjtC11NzEgr4wyiVju1xwg+bh12H+7843iUZICMYMf5wBR07I3mI0d0sKsaF6luLSJlBTPF3hCm2y2h4TLQ+ctpEpA2R3WamY3gjsmlU/De9rVSLmh4P5lp+xb80WtCdYvzDoiyCt5gM50YmBPMWzRNw+vFE9FnnD8HPt5hDgtoof/f90Fwa4nzuzigmAoB714NnO19avhH7AGtfDkgAA+J0TrkfM6dsTy1j8sis1zLqdxQR5HxFV9uDTzPup32eNE4qRxny3JWUomXaSF9AAADo6gekcqcRX8DqV4+S+SCeoUrxc+LfoB5IIGa5RCCeBt2ES9BEQOEp8zUiMUMo70obE0rXlz8ifgAAkdYpB7MLOb4O6qIJTt0iaRK2ELWtVR6Qpeu/0AULcsFgAAAAA)

**mark()**：把当前的position赋值给mark

	public  final  Buffer mark()  {	    
	 	mark = position;	    
	 	return  this;	    
	}
    

**reset()**：把mark值还原给position

	public  final  Buffer reset()  {    
	 	int m = mark;    
	 	if  (m <  0)    
	 	throw  new  InvalidMarkException();    
	 	position = m;    
	 	return  this;    
	}
    

**clear()**：一旦读完Buffer中的数据，需要让Buffer准备好再次被写入，clear会恢复状态值，但不会擦除数据。

	public  final  Buffer clear()  {    
	 	position =  0;    
	 	limit = capacity;    
	 	mark =  -1;    
	 	return  this;    
	}
    

**flip()**：Buffer有两种模式，写模式和读模式，flip后Buffer从写模式变成读模式。

	public  final  Buffer flip()  {    
	 	limit = position;	    
	 	position =  0;	    
	 	mark =  -1;	    
	 	return  this;    
	}
    

**rewind()**：重置position为0，从头读写数据。

	public  final  Buffer rewind()  {    
	 	position =  0;    
	 	mark =  -1;    
	 	return  this;    
	}
    

目前Buffer的实现类有以下几种：

*   ByteBuffer
    
*   CharBuffer
    
*   DoubleBuffer
    
*   FloatBuffer
    
*   IntBuffer
    
*   LongBuffer
    
*   ShortBuffer
    
*   MappedByteBuffer
    

![](data:image;base64,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)

### ByteBuffer

> A byte buffer，extend from Buffer

ByteBuffer的实现类包括"HeapByteBuffer"和"DirectByteBuffer"两种。

#### HeapByteBuffer

	public  static  ByteBuffer allocate(int capacity)  {    
	 	if  (capacity <  0)	    
	 	throw  new  IllegalArgumentException();	    
	 	return  new  HeapByteBuffer(capacity, capacity);    
	}
	
	HeapByteBuffer(int cap,  int lim)  {    
		super(-1,  0, lim, cap,  new  byte\[cap\],  0);    
	}
    

HeapByteBuffer通过初始化字节数组hd，在虚拟机堆上申请内存空间。

#### DirectByteBuffer

	public static ByteBuffer allocateDirect(int capacity) {
		return new DirectByteBuffer(capacity);
	}
	
	DirectByteBuffer(int cap) {
		super(-1, 0, cap, cap);
		boolean pa = VM.isDirectMemoryPageAligned();
		int ps = Bits.pageSize();
		long size = Math.max(1L, (long) cap + (pa ? ps : 0));
		Bits.reserveMemory(size, cap);
		long base = 0;
		try {
			base = unsafe.allocateMemory(size);
		} catch (OutOfMemoryError x) {
			Bits.unreserveMemory(size, cap);
			throw x;
		}
		unsafe.setMemory(base, size, (byte) 0);
		if (pa && (base % ps != 0)) {
			// Round up to page boundary
			address = base + ps - (base & (ps - 1));
		} else {
			address = base;
		}
		cleaner = Cleaner.create(this, new Deallocator(base, size, cap));
		att = null;
	}
    

DirectByteBuffer通过unsafe.allocateMemory在物理内存中申请地址空间（非jvm堆内存），并在ByteBuffer的address变量中维护指向该内存的地址。 unsafe.setMemory(base, size, (byte) 0)方法把新申请的内存数据清零。

### Channel

> A channel represents an open connection to an entity such as a hardware device, a file, a network socket, or a program component that is capable of performing one or more distinct I/O operations, for example reading or writing.

NIO把它支持的I/O对象抽象为Channel，Channel又称“通道”，类似于原I/O中的流（Stream），但有所区别： 

1、流是单向的，通道是双向的，可读可写。 

2、流读写是阻塞的，通道可以异步读写。 

3、流中的数据可以选择性的先读到缓存中，通道的数据总是要先读到一个缓存中，或从缓存中写入，如下所示：

![](data:image;base64,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)

目前已知Channel的实现类有：

*   FileChannel
    
*   DatagramChannel
    
*   SocketChannel
    
*   ServerSocketChannel
    

#### FileChannel

> A channel for reading, writing, mapping, and manipulating a file. 一个用来写、读、映射和操作文件的通道。

FileChannel的read、write和map通过其实现类FileChannelImpl实现。

##### read实现

	public int read(ByteBuffer dst) throws IOException {
		ensureOpen();
		if (!readable)
			throw new NonReadableChannelException();
		synchronized (positionLock) {
			int n = 0;
			int ti = -1;
			try {
				begin();
				ti = threads.add();
				if (!isOpen())
					return 0;
				do {
					n = IOUtil.read(fd, dst, -1, nd);
				} while ((n == IOStatus.INTERRUPTED) && isOpen());
				return IOStatus.normalize(n);
			} finally {
				threads.remove(ti);
				end(n > 0);
				assert IOStatus.check(n);
			}
		}
	}    

FileChannelImpl的read方法通过IOUtil的read实现：

	static  int read(FileDescriptor fd,  ByteBuffer dst,  long position,
					 NativeDispatcher nd)  IOException  {
		if  (dst.isReadOnly())
			throw  new  IllegalArgumentException("Read-only buffer");
		if  (dst instanceof  DirectBuffer)
			return readIntoNativeBuffer(fd, dst, position, nd);
		// Substitute a native buffer
		ByteBuffer bb =  Util.getTemporaryDirectBuffer(dst.remaining());
		try  {
			int n = readIntoNativeBuffer(fd, bb, position, nd);
			bb.flip();
			if  (n >  0)
				dst.put(bb);
			return n;
		}  finally  {
			Util.offerFirstTemporaryDirectBuffer(bb);
		}
	}
    

通过上述实现可以看出，基于channel的文件数据读取步骤如下： 

1、申请一块和缓存同大小的DirectByteBuffer bb。 

2、读取数据到缓存bb，底层由NativeDispatcher的read实现。 

3、把bb的数据读取到dst（用户定义的缓存，在jvm中分配内存）。 

**read方法导致数据复制了两次**。

##### write实现

	public  int write(ByteBuffer src)  throws  IOException  {
		ensureOpen();
		if  (!writable)
			throw  new  NonWritableChannelException();
		synchronized  (positionLock)  {
			int n =  0;
			int ti =  -1;
			try  {
				begin();
				ti = threads.add();
				if  (!isOpen())
					return  0;
				do  {
					n =  IOUtil.write(fd, src,  -1, nd);
				}  while  ((n ==  IOStatus.INTERRUPTED)  && isOpen());
				return  IOStatus.normalize(n);
			}  finally  {
				threads.remove(ti);
				end(n >  0);
				assert  IOStatus.check(n);
			}
		}
	}
    

和read实现一样，FileChannelImpl的write方法通过IOUtil的write实现：

	static  int write(FileDescriptor fd,  ByteBuffer src,  long position,
					  NativeDispatcher nd)  throws  IOException  {
		if  (src instanceof  DirectBuffer)
			return writeFromNativeBuffer(fd, src, position, nd);
		// Substitute a native buffer
		int pos = src.position();
		int lim = src.limit();
		assert  (pos <= lim);
		int rem =  (pos <= lim ? lim - pos :  0);
		ByteBuffer bb =  Util.getTemporaryDirectBuffer(rem);
		try  {
			bb.put(src);
			bb.flip();
			// Do not update src until we see how many bytes were written
			src.position(pos);
			int n = writeFromNativeBuffer(fd, bb, position, nd);
			if  (n >  0)  {
				// now update src
				src.position(pos + n);
			}
			return n;
		}  finally  {
			Util.offerFirstTemporaryDirectBuffer(bb);
		}
	}
    

通过上述实现可以看出，基于channel的文件数据写入步骤如下： 

1、申请一块DirectByteBuffer，bb大小为byteBuffer中的limit - position。 

2、复制byteBuffer中的数据到bb中。 

3、把数据从bb中写入到文件，底层由NativeDispatcher的write实现，具体如下：

	private  static  int writeFromNativeBuffer(FileDescriptor fd, ByteBuffer bb,  long position,  NativeDispatcher nd)
			throws  IOException  {
		int pos = bb.position();
		int lim = bb.limit();
		assert  (pos <= lim);
		int rem =  (pos <= lim ? lim - pos :  0);
		int written =  0;
		if  (rem ==  0)
			return  0;
		if  (position !=  -1)  {
			written = nd.pwrite(fd,
					((DirectBuffer)bb).address()  + pos,
					rem, position);
		}  else  {
			written = nd.write(fd,  ((DirectBuffer)bb).address()  + pos, rem);
		}
		if  (written >  0)
			bb.position(pos + written);
		return written;
	}
    

write方法也导致了数据复制了两次

### Channel和Buffer示例

	File file =  new  RandomAccessFile("data.txt",  "rw");
	FileChannel channel = file.getChannel();
	ByteBuffer buffer =  ByteBuffer.allocate(48);
	int bytesRead = channel.read(buffer);
	while  (bytesRead !=  -1)  {
		System.out.println("Read "  + bytesRead);
		buffer.flip();
		while(buffer.hasRemaining()){
			System.out.print((char) buffer.get());
		}
		buffer.clear();
		bytesRead = channel.read(buffer);
	}
	file.close();
    

注意buffer.flip() 的调用，首先将数据写入到buffer，然后变成读模式，再从buffer中读取数据。

### 总结

通过本文的介绍，希望大家对Channel和Buffer在文件读写方面的应用和内部实现有了一定了解，努力做到不被一叶障目。










