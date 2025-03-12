# ExamLast


# Подготовка


### ER диаграмма - МОДУЛЬ 1

1. Visio
2. Новый документ
3. Вставка -> Размер -> A3
4. Текстовое поле - Название гостиницы и ER диаграмма в 3-НФ
5. Дополнительные фигуры -> Программы и базы данных -> Базы данных -> Нотация IDEF1X
6.  Заполняем 
	- Clients
	- Orders
	- Rooms
	- Cards
	- Users
	- Roles
7.  Добавляем связи
8. Сохраняем в PDF
### Таблица Excell и БД - МОДУЛЬ 2

1. Excell
	- Переименовываем поля в Excell Документе
	- Заполняем новые поля
2. Создаем бд со своей фамилией 
3. ПКМ по БД -> Задачи -> Импорт -> Источник данных (Microsoft excell) - добавляем файл, версия excell -> Назначение (Microsoft OLE DB Provider SQL Server) - Проверка подлинности SQL Server -> Далее -> Далее -> Далее - Финиш
4. Переименовывем таблицу 
5. Правим тип данных ( id - int, Задать  первичный ключ, спецификация идентификатора - да - для счетчика) 
6. Сохраняем
7. Создаем таблицы и записываем все поля (Задаем первичный ключ, сохраняем таблицу, задаем название во множественном числе (s) )
8. Пкм по диаграммы баз данных -> Создать -> Да Добавить все таблицы -> Создаем связи (Ключик тащим к полю) -> связываем по одинаковым полям -> спецификация INSERT и UPD -> правило обновления и удаления - каскадно
9. Вводим имя диаграммы и сохраняем 
10.  Делаем запрос SELECT COUNT(CASE WHEN RoomStatus = 'занят' THEN 1 END)* 100.0/COUNT( * ) AS ' percent' FROM Rooms
11. Сохраняем запрос в папку

### Разработка - МОДУЛЬ 3

1. Создаем проект WPF .NET framework
2. Создаем внутри такие директории
	- Data
		- 
	- Pages
		- AdminPage
		- ManagerPage
	- Windows
		- Desktop - Окно (добавить окно)
		- Authorization - Окно
		- ChangePassword - Окно
		- AddUser - Окно
		- ChangeUser - Окно
3. Удаляем MainWindow -> App.xaml -Windows/Desktop.xaml

	#### Пишем код

1.  Desktop
```C#
	new Authorization().ShowDialog();
	StaticObjects.desktopFrame = DesktopFrame;
```
2. Добавляем все нужные элементы для окон + прописываем имена x:Name="имя" и изменяем Title + центрируем
3. В Desktop добавляем новый фрейм и меняем название
4. На страницу админа в DataGrid -  
```c#
	x:Name="DtGrid" DataContext="{Binding}"
```

5. Добавляем контекст БД -> Добавить -> Данные -> ADO.NET EDM -> Имя -> CodeFirst из БД -> Создать соединение -> Проврека подлинности SQL Server -> Шифрование Optional -> Проверить подключение -> Да включить конфиденциальные данные в строку подключения -> Убираем не нужные таблицы -> Галочка формировать имена объектов 
6. После создания переносим все по директориям Models и Context
7. Создаем файл StaticObjects 
```cs
    public static User user;
```
8. Записываем фрейм в StaticObject 
```cs
 public static Frame desktopFrame;
```
9.  Добавляем две роли Админа и Менеджера и создаем двух пользователей 

- Авторизация 
	- Обозначаем Контекст 
```cs
	TestContext db = new TestContext();
	int attemptCount = 0;
```
```cs
	if (string.IsNullOrEmpty(TbLogin.Text))
	{
	    MessageBox.Show("Вы не ввели логин");
	    TbLogin.Focus();
	}
	else if (string.IsNullOrEmpty(TbPassword.Text))
	{
	    MessageBox.Show("Вы не ввели Пароль");
	    TbLogin.Focus();
	}
	else
	{
	    User user = db.Users.Where(u=>u.UserName == TbLogin.Text).FirstOrDefault();
	    if (user == null) {
	        MessageBox.Show("Такого пользователя нет");
	        return;
	    }
	    if((DateTime.Now - (DateTime)user.LastLoginDate).Days > 31){
	        MessageBox.Show("Пользователь заблокирован обратитесь к администратору");
	        return;
	    }
	    if(attemptCount>=3)
	    {
		    user.Blocked = true;
			db.SaveChanges();
	        MessageBox.Show("Попытки входа исчерпаны ваш аккаунт заблокирован");
	        return;
	    }
	    if (user.Blocked == true) {
		    MessageBox.Show("Пользователь заблокирован, обратитесь к администратору");
		    return;
		}
		if (user.Password.trim() == TbPassword.Text) { 
	        StaticObjects.user = user;
	        if (!user.PasswordChanged)
	        {
	            ChangePassword changePassword = new ChangePassword();
	            changePassword.user = user;
	            changePassword.ShowDialog();
	        }
	        else 
	        { 
	            user.LastLoginDate = DateTime.Now;
	            db.SaveChanges();
	            MessageBox.Show("Вы успешно авторизировались");
	            if (StaticObjects.user.RoleId == 1) { 
					StaticObjects.desktopFrame.Navigate(new AdminPage());
					this.Close(); 
	            }
	            else if (StaticObjects.user.RoleId == 2)
	            {
					StaticObjects.desktopFrame.Navigate(new ManagerPage());
					this.Close();
	            }
	        }
	    }
	    else
	    {
	        attemptCount++;
	        MessageBox.Show($"Вы ввели неправильный пароль, осталось {3 - attemptCount} попытки входа");
	        return;
	    }
	}
```

- Смена пароля 
```cs
public User user = new User();
TestContext db = new TestContext();
```
```cs
if (string.IsNullOrEmpty(TbPassword.Text)) {
    MessageBox.Show("Вы не ввели старый пароль");
    TbPassword.Focus();
    return;
}
if (string.IsNullOrEmpty(TbNewPassword.Text))
{
    MessageBox.Show("Вы не ввели новый пароль");
    TbNewPassword.Focus();
    return;
}
if (string.IsNullOrEmpty(TbApprove.Text))
{
    MessageBox.Show("Вы не ввели подтверждение пароля");
    TbApprove.Focus();
    return;
}
if (TbPassword.Text == TbNewPassword.Text)
{
    MessageBox.Show("Старый пароль не может совпадать с новым паролем");
    TbNewPassword.Focus();
    return;
}
if (TbPassword.Text != user.Password.Trim())
{
    MessageBox.Show("Вы ввели неверный старый пароль");
    TbPassword.Focus();
    return;
}
if (TbNewPassword.Text != TbApprove.Text) {

    MessageBox.Show("Пароли не совпадают");
    TbApprove.Focus();
}
else
{
    user.Password = TbNewPassword.Text;
    user.PasswordChanged = true;
    db.SaveChanges();
    MessageBox.Show("Теперь вы можете войти под новым паролем");
    this.Close();  
}
```

- Страница Администратора 
```cs
        	TestContext db = new TestContext();
		public AdminPage()
		{
		    InitializeComponent();
			StaticObjects.dataGrid = DtGrid;
			StaticObjects.dataGrid.ItemsSource = db.Users.ToList();
		}


		private void BtnAdd_Click_1(object sender, RoutedEventArgs e)
		{
		    new AddUser().ShowDialog();
		}

		private void BtnChange_Click(object sender, RoutedEventArgs e)
		{
		    User user = DtGrid.SelectedItem as User;
		    if (user != null)
			{
		        ChangeUser changeUser = new ChangeUser(user);
		        changeUser.ShowDialog();
		    }
		    else {
		        MessageBox.Show("Не выбран пользователь");
		     }
		}

		private void BtnDelete_Click(object sender, RoutedEventArgs e)
		{
    			User selectedUser = DtGrid.SelectedItem as User;

			if (selectedUser == null)
    			{
        			MessageBox.Show("Выберите пользователя для удаления.");
        			return;
    			}

    			if (StaticObjects.user.UserId == selectedUser.UserId)
    			{
        			MessageBox.Show("Для удаления данного пользователя необходимо войти в систему под другим пользователем.");
        			return;
    			}
    			else
			{
        			if (MessageBox.Show("Удалить запись?", "Удаление записи", MessageBoxButton.YesNo, MessageBoxImage.Question) == MessageBoxResult.Yes)
        			{
            				using (var db = new TestContext()) // Создаем новый контекст
            				{
                				var userToDelete = db.Users.Find(selectedUser.UserId); // Ищем пользователя в БД

                				if (userToDelete != null)
                				{
                    					db.Users.Remove(userToDelete);
                    					db.SaveChanges();
                    					StaticObjects.dataGrid.ItemsSource = db.Users.ToList(); // Обновляем DataGrid
						}
	               				 else
						{
                    					MessageBox.Show("Пользователь не найден в базе данных.");
						}
           				 }
        			}
    			}
		}
```

- Добавление пользователя
```cs
TestContext db = new TestContext();
public AddUser()
{
    InitializeComponent();
    CmbBox.ItemsSource = db.Roles.ToList();
    CmbBox.DisplayMemberPath = "RoleName";
    CmbBox.SelectedValuePath = "RoleId";
}


private void BtnAdd_Click(object sender, RoutedEventArgs e)
{
    try
    {
        if (string.IsNullOrEmpty(TbLogin.Text) || string.IsNullOrEmpty(TbPassword.Text) || string.IsNullOrEmpty(CmbBox.Text))
        {
            MessageBox.Show("Заполните все поля");
            return;
        }
        else if (db.Users.Where(u => u.UserName == TbLogin.Text).ToList().Count > 0)
        {
            MessageBox.Show("Такой пользователь уже существует");
            return;
        }
        else
        {
            db.Users.Add(new User
            {
                UserName = TbLogin.Text,
                Password = TbPassword.Text,
                RoleId = Convert.ToInt32(CmbBox.SelectedValue),
                LastLoginDate = DataPick.DisplayDate,
                PasswordChanged = (bool)ChkBoxPass.IsChecked,
                Blocked = (bool)ChkBoxBlock.IsChecked
            });
            db.SaveChanges();
        }

        StaticObjects.dataGrid.ItemsSource = db.Users.ToList();
        this.Close();
    }
    catch (Exception ex) { 
        MessageBox.Show(ex.Message);
    }
}
```

- Редактирование пользователя
```cs
TestContext db = new TestContext();
public User user;
public ChangeUser(User user)
{
    this.user = user;
    InitializeComponent();

    CmbBox.ItemsSource = db.Roles.ToList();
    CmbBox.DisplayMemberPath = "RoleName";
    CmbBox.SelectedValuePath = "RoleId";

    TbLogin.Text = user.UserName;
    TbPassword.Text = user.Password;
    CmbBox.Text = db.Roles.Find(user.RoleId).RoleName;
    DataPick.Text = user.LastLoginDate.ToString();
    ChkBoxPass.IsChecked = user.PasswordChanged;
    ChkBoxBlock.IsChecked = user.Blocked;
}

private void BtnChangeUser_Click(object sender, RoutedEventArgs e)
{
    try
    {
        if (string.IsNullOrEmpty(TbLogin.Text) || string.IsNullOrEmpty(TbPassword.Text) || string.IsNullOrEmpty(CmbBox.Text))
        {
            MessageBox.Show("Заполните все поля");
            return;
        }
        else
        {
            User editUser = db.Users.Find(this.user.UserId);
            editUser.UserName = TbLogin.Text;
            editUser.Password = TbPassword.Text;
            editUser.RoleId = Convert.ToInt32(CmbBox.SelectedValue);
            editUser.LastLoginDate = DataPick.DisplayDate;
            editUser.PasswordChanged = (bool)ChkBoxPass.IsChecked;
            editUser.Blocked = (bool)ChkBoxBlock.IsChecked;
            db.SaveChanges();
        }

        StaticObjects.dataGrid.ItemsSource = db.Users.ToList();
        this.Close();
    }
    catch (Exception ex)
    {
        MessageBox.Show(ex.Message);
    }
}
```
