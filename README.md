# Проектирование высоконагруженных систем. Курсовой проект



<h2>1. Тема</h2>

YouTube на русскоязычную аудиторию



<h2>2. Определение возможного диапазона нагрузок подобного проекта</h2>

На видеохостингах основная нагрузка приходится на просмотр видео, поэтому необходимо посчитать сколько часов видео российские пользователи смотрят в YouTube.

По данным [Google](https://blog.youtube/news-and-events/you-know-whats-cool-billion-hours) в день на YouTube просматривается более одного миллиарда видео. Эти просмотры генерируются глоабальной аудиторией — MAU более двух миллиардов человек(исходя из [пресс-релиза Google](https://www.youtube.com/intl/ru/about/press/)). Таким образом один человек в месяц смотрит в среднем 15 видео на youtube. 

По данным сайта [businessofapps.com](https://www.businessofapps.com/data/youtube-statistics/) YouTube **российская аудитория youtube** составляет 5% от глобальной аудитории, что равняется примерно **50 000 000 пользователей**.

Следовательно **в месяц в России на YouTube ** 15 * 50 000 000 = **750 000 000 просмотров видео**.

Если учесть, что одно видео в YouTube длиться примерно 30 минут, то получается, что всего **в Росиии генерируется**  750 000 000 * 0.5 = **375 000 000 часов просмотра видео в месяц** **или** 375 000 000 / 30 = **12 500 000 часов в день**.

Общее колличество видео на Youtube равно 1 300 000 000, следовательно для **российской аудитории это значение будет** 1 300 000 000 * 0.05 = **65 000 000 видео**.



<h2>3. Выбор планируемой нагрузки</h2>

- MAU российского сегмента youtube примерно равен 50 млн пользователей
- Клличество просмотров в месяц видео в месяц равно 750 млн
- Колличество часов просмотра в месяц: 375 млн
- Общее колличество видео: 65 млн

Если грубо посчитать DAU = MAU/30 = 50 000 000 / 30 ≈ 1 700 000



Заходя на сайт пользователь выбирает видео из списка, после чего переходит на страницу просмотра. По статистике решение о том, чтобы продолжить просмотр пользователь принимает за 10 секунд, поэтому первые 10 секунд каждого видео являются самым просматриваемым местом. 

Допустим, что пользователь досматривает до конца каждое третье видео, которое начинает смотреть. Тогда в среднем, перед тем, как начать просмотр видео, пользователь посещает 4 страницы: главную и 3 перехода между разными видео.

С главной страницы на сервер уходит порядка 50 запросов, со страниц с видео порядка 100 запросов. То есть в среднем до начала просмотра видео пользователь генерирует порядка 350 запросов. Умножаем на колличество пользователей и делим на колличество секунд в сутках. **Получаем 7000 rps.**

![формула](https://latex.codecogs.com/svg.latex?\frac{(50%20+%203%20*%20100)%20*%201700000}{\24%20*%2060%20*%2060}%20%20=%207000rps)

<h2>4. Логическая схема базы данных</h2>

Модели проекта:

- Пользователь

- Видео

- Лайк

- Комментарий

- Плейлист

  

База данных намеренно денормализована так, чтобы в моделе видео храниться пользователь, который его загрузил, комментарий хранит модель пользователя, который его написал, плейлист хранит в себе информацию о создавшем его пользователе. Лайк же, например, хранит исключительно id объектов, а не их полные модели.

В схеме базы данных не учтены всевозможные статистические данные для подбора релеватных видео для пользователя.







![youtube_highload](https://raw.githubusercontent.com/BoldinDmitry/highload_project/main/files/youtube_highload.jpg)



<h2>5. Физическая система хранения </h2>

Для хранения сессий будем использовать in-memory Redis. Благодаря тому, что Redis является in-memory базой данных, то доступ к определенной записи будет практически равен доступу к оперативной памяти. Помимо свой быстроты Redis поддерживает master-slave репликацию из коробки. 

Для хранения всех сессий пользователей(50 млн) согласно [статье](https://medium.com/@lucasmagnum/redistip-estimate-the-memory-usage-for-repeated-keys-in-redis-2dc3f163fdab) нам примерно потребуется: 50 000 000 * 220 = 11 000 000 000 байт = 11 GB

