## TODO Application using containers
create a versatile ToDo app using (Item/Model) based containers
- [Introduction](#INTRO)
- [ItemBasedModel](#ItemBaseModel)
- [MVCModel](#MVCModel)

<div id = "back"></div>


  
## **Introduction** 

<a name="INTRO"></a>

**Widgets** are the primary elements for creating user interfaces in Qt. Widgets can display data and status information, receive user input, and provide a container for other widgets that should be grouped together. A widget that is not embedded in a parent widget is called a window.

A **Container** is a holder object that stores a collection of other objects(its elements). They are implemented as class templates, which allows a great flexibility in the types supported as elements. It replicate structures very commonly used in programming : dynamic arrays(vector), queues, stacks, heaps, linked list, trees, maps... We will check three basic containers which are list, tree and table, and for each one we will check two variant Item Based and Model Based.

The **QTableWidget** class provides an item-based table view with a default model. Table widgets provide standard table display facilities for applications. The items in a QTableWidget are provided by QTableWidgetItem.

A **QListWidget** is a convenience class that provides a list view similar to the one supplied by QListView, but with a classic item-based interface for adding and removing items. QListWidget uses an internal model to manage each QListWidgetItem in the list.

The **QTreeWidget class** is a convenience class that provides a standard tree widget with a classic item-based interface similar to that used by the QListView class in Qt 3. This class is based on Qt's Model/View architecture and uses a default model to hold items, each of which is a QTreeWidgetItem.

The **QTableView** class provides a default **Model/View** implementation of a table view. It implements a table view that displays items from a model. This class is used to provide standard tables that were previously provided by the QTable class, but using the more flexible approach provided by Qt's model/view architecture.The QTableView class is one of the Model/View Classes and is part of Qt's model/view framework. It implements the interfaces defined by the QAbstractItemView class to allow it to display data provided by models derived from the QAbstractItemModel class.

![Image](/Items.png)

**Model–view–controller (MVC)** is a software design pattern commonly used for developing user interfaces that divide the related program logic into three interconnected elements. MVC consists of three kinds of objects. The Model is the application object, the View is its screen presentation, and the Controller defines the way the user interface reacts to user input. Before MVC, user interface designs tended to lump these objects together. MVC decouples them to increase flexibility and reuse.

![Image](/mvc.png)

 >**In This Zip you will have the project** [Homework4.zip]() 
 
 [(**Back to top**)](#back)
 
### ToDoApp

We will create an application to manage tasks. It will have all the features of main application such as menus, actions and toolbar. The application stores an archive of all the pending and finished tasks.

![Image](/todoappp.png)

### Item Based Model

<a name="ItemBasedModel"></a>

In the Item Based Model, We wrote the code for the graphical and set of actions , now we will write a set of basic functionality. we will start with the connections made for our application:

1.we add the function for the newTask action,we created a Dialog for the user to add tasks, for that, first we created a Form Class, we use the designer and we obtain the form of AddNew, in addition we added some methods to get the content of our line Edit, checkBox, comboBox and the Date Edit. Here is the Form of Add new task:

![Image](/dialogui.png)

first we add those lines to the header file of the dialog class :
```javascript
class Dialog1 : public QDialog
{
    Q_OBJECT

public:
    explicit Dialog1(QWidget *parent = nullptr);
    ~Dialog1();
    void taskDialog();
    bool isChecked();
    void setdate(int a,int m, int j);
    void task(QString t);
    void tag(QString a);
    void finished(bool f);
    QDate getDate();
    QString getText();

private:
    Ui::Dialog1 *ui;
};
```
and in the cpp file we implement our methods:
```javascript
Dialog1::Dialog1(QWidget *parent) :
    QDialog(parent),
    ui(new Ui::Dialog1)
{
    ui->setupUi(this);
    QDate date =QDate::currentDate();

    ui->dateEdit->setMinimumDate(date);

    ui->dateEdit->setDate(date);
}

Dialog1::~Dialog1()
{
    delete ui;
}
void Dialog1::setdate(int a, int m, int j){

    QDate d;


    d.setDate(a,m,j);

    ui->dateEdit->setDate(d);

}
void Dialog1::task(QString t){
    ui->lineEdit->setText(t);
}
void Dialog1::tag(QString a){
    ui->comboBox->setCurrentText(a);
}

void Dialog1::finished(bool f){
    ui->checkBox->setChecked(f);

}
bool Dialog1::isChecked(){
    if(ui->checkBox->isChecked()){
        return true;
    }
    return false;
}
QDate Dialog1::getDate(){
    return ui->dateEdit->date();
}
QString Dialog1::getText(){
    QString a= ui->lineEdit->text()+"Due :" + ui->dateEdit->text() + "Tag :" + ui->comboBox->currentText() + "";
    return a;
}
```

In addition,we have declared a private slot called Newtask 

```javascript
private slots:
    void NewTask();
```
We created a connexion between the newtask action and the method : 


```javascript
connect(newtask,&QAction::triggered,this,&toDoApp::NewTask);
```

Then, we implement the function in the cpp file:

```javascript
void toDoApp::NewTask()
{

    Dialog1 dialog;
    auto replu = dialog.exec();
    if(replu==Dialog1::Accepted){
        QString text = dialog.getText();
        if(dialog.getDate()==QDate::currentDate() && !dialog.isChecked()){
            QIcon TodayIcon(":/Todayicon.png");

            ui->pesistent->addItem(new QListWidgetItem(TodayIcon,text));
        }
        else if(dialog.getDate()!=QDate::currentDate() && !dialog.isChecked()){
            QIcon pendingIcon(":/pending_icon.png");

            ui->Pending->addItem(new QListWidgetItem(pendingIcon,text));
        }
        else if(dialog.isChecked()){
            QIcon completedIcon(":/completed_icon.png");

            ui->completed->addItem(new QListWidgetItem(completedIcon,text));
        }
    }


}
```
2.For adding a new task, the user choose if the task is for today , pending or finished 
   First, we declare three slots in the header file for the three lists view
   ```javascript
private slots:
    void Firsttask();
    void secondtask();
    void thirdtask();

```
Then we implement the function in the cpp file :
    The function first task is for the tasks of today
   ```javascript
   void toDoApp::Firsttask(){
     Dialog1 dialog;
    int i=1;
    QString tasks;
    QListWidgetItem *a = ui->pesistent->currentItem();
    QStringList list = a->text().split(QRegularExpression("\\W+"),Qt::SkipEmptyParts);
    tasks+= list[0];
    while(list[i]!= "Due"){
        tasks+= " " + list[i];
        i++;

    }
    dialog.setdate(list[i+3].toInt(),list[i+1].toInt(),list[i+2].toInt());
    dialog.task(tasks);
    dialog.tag(list[i+5]);
    auto reply = dialog.exec();

    if(reply==  Dialog1::Accepted){

        QString text= dialog.getText();
        if(dialog.getDate()== QDate::currentDate() && !dialog.isChecked()){
            QIcon TodayIcon(":/Todayicon.png");

            ui->pesistent->addItem(new QListWidgetItem(TodayIcon,text));
        }else if(dialog.getDate()!= QDate::currentDate() && !dialog.isChecked()){
            QIcon pendingIcon(":/pending_icon.png");

            ui->Pending->addItem(new QListWidgetItem(pendingIcon,text));

        }else if(dialog.isChecked()){
            QIcon completedIcon(":/completed_icon.png");

            ui->completed->addItem(new QListWidgetItem(completedIcon,text));
        }
        delete a;
    }

}
   ```
 The function second task is for the pending tasks
 ```javascript
void toDoApp::secondtask(){
    Dialog1 dialog;
    int i=1;
    QString tasks;
    QListWidgetItem *a = ui->Pending->currentItem();
    QStringList list = a->text().split(QRegularExpression("\\W+"),Qt::SkipEmptyParts);
     tasks+= list[0];
    while(list[i]!= "Due"){
        tasks+= " " + list[i];
        i++;

    }
    dialog.setdate(list[i+3].toInt(),list[i+1].toInt(),list[i+2].toInt());
    dialog.task(tasks);
    dialog.tag(list[i+5]);
    auto reply = dialog.exec();

    if(reply== Dialog1::Accepted){

        QString text= dialog.getText();
        if(dialog.getDate()== QDate::currentDate() && !dialog.isChecked()){
            QIcon TodayIcon(":/Todayicon.png");

            ui->pesistent->addItem(new QListWidgetItem(TodayIcon,text));
        }else if(dialog.getDate()!= QDate::currentDate() && !dialog.isChecked()){
            QIcon pendingIcon(":/pending_icon.png");

            ui->Pending->addItem(new QListWidgetItem(pendingIcon,text));

        }else if(dialog.isChecked()){
            QIcon completedIcon(":/completed_icon.png");

            ui->completed->addItem(new QListWidgetItem(completedIcon,text));
        }
        delete a;
    }
}

```
 The function third task is for the finished tasks
 
 ```javascript
void toDoApp::thirdtask(){
    Dialog1 dialog;
    int i=1;
    QString tasks;
    QListWidgetItem *a = ui->completed->currentItem();
    QStringList list = a->text().split(QRegularExpression("\\W+"),Qt::SkipEmptyParts);
     tasks+= list[0];
    while(list[i]!= "Due"){
        tasks+= " " + list[i];
        i++;

    }
    dialog.setdate(list[i+3].toInt(),list[i+1].toInt(),list[i+2].toInt());
    dialog.task(tasks);
    dialog.tag(list[i+5]);
    auto reply = dialog.exec();

    if(reply== Dialog1::Accepted){

        QString text= dialog.getText();
        if(dialog.getDate()== QDate::currentDate() && !dialog.isChecked()){
            QIcon TodayIcon(":/Todayicon.png");

            ui->pesistent->addItem(new QListWidgetItem(TodayIcon,text));
        }else if(dialog.getDate()!= QDate::currentDate() && !dialog.isChecked()){
            QIcon pendingIcon(":/pending_icon.png");

            ui->Pending->addItem(new QListWidgetItem(pendingIcon,text));

        }else if(dialog.isChecked()){
            QIcon completedIcon(":/completed_icon.png");

            ui->completed->addItem(new QListWidgetItem(completedIcon,text));
        }
        delete a;
    }
}

```
We add the connexion of these three slots :
```javascript
    connect(ui->pesistent,SIGNAL(itemDoubleClicked(QListWidgetItem*)),this,SLOT(Firsttask()));
    connect(ui->Pending,SIGNAL(itemDoubleClicked(QListWidgetItem*)),this,SLOT(secondtask()));
    connect(ui->completed,SIGNAL(itemDoubleClicked(QListWidgetItem*)),this,SLOT(thirdtask()));

```
3.We added a closeEvent that save the data in a file 
  First, we declare the slots in the header file
  
  ```javascript
protected:
    void closeEvent(QCloseEvent *e) override;

```
Then we implement the function in the cpp file :
  ```javascript
void toDoApp::closeEvent(QCloseEvent *e){

    QFile file("/Users/hp/Desktop/save.txt");
    if (file.open(QIODevice::ReadWrite| QIODevice::Text)){

        QTextStream out(&file);
        for (int i=0;i<ui->pesistent->count() ;i++ ) {
            out<< "1"<< ui->pesistent->item(i)->text() << Qt::endl;
        }
        for (int i=0;i<ui->Pending->count() ;i++ ) {
            out<< "2"<< ui->pesistent->item(i)->text() << Qt::endl;
        }
        for (int i=0;i<ui->completed->count() ;i++ ) {
            out<< "3"<< ui->pesistent->item(i)->text() << Qt::endl;
        }
        file.close();
    }
}

```
4.For open the previous data , we added some line to open our file.txt
```javascript
QFile file("/Users/hp/Desktop/save.txt");
    if (!file.open(QIODevice::ReadOnly | QIODevice::Text))
        return;

    while (!file.atEnd()) {
        QString line = file.readLine();
        if(line.at(0)== "1"){
            ui->pesistent->addItem(line.mid(1,line.size()));
        }else if(line.at(0)== "2"){
             ui->Pending->addItem(line.mid(1,line.size()));
        }else if(line.at(0)== "3"){
            ui->completed->addItem(line.mid(1,line.size()));

        }
    }

```
5. we added a slots called Pending slot and completed slot
First, we declare the slots in the header file
```javascript
private slots:
    void PendingSlot();
private slots:
    void CompletedSlot();

```
Then we implement the function in the cpp file :

```javascript
void toDoApp::PendingSlot(){
    if(ui->Pending->isVisible()){
        ui->Pending->hide();
}else{
   ui->Pending->show();
    }
}

void toDoApp::CompletedSlot(){
    if(ui->completed->isVisible()){
        ui->completed->hide();
}else{
   ui->completed->show();
    }
}
```
We add the connexion of these slots to enable the actions to show or hide our listWidget
```javascript
connect(pending,&QAction::triggered,this,&toDoApp::PendingSlot);
connect(completed,&QAction::triggered,this,&toDoApp::CompletedSlot);
```
6. we added a slots called quit to close the window, and the about and aboutQt slots:

First, we declare the slots in the header file
```javascript
private slots:
    void quit();
    void aboutslot();
    void aboutQtslot();
 
```
Then we implement the functions in the cpp file 
```javascript
void toDoApp::quit(){
    auto reply = QMessageBox::question(this, "Exit","Do you really want to quit?");
    if(reply == QMessageBox::Yes)
        qApp->exit();
}
void toDoApp::aboutslot(){
    QMessageBox::about(this,"about","to do app is an app to manage tasks");;
}
void toDoApp::aboutQtslot(){
    QMessageBox::aboutQt(this, "Your Qt");
}
```
![Image](/exitpic.png)

![Image](/about.png)

![Image](/aboutQt.png)
7. we added a slot called clear for delete the content of our list views :
     First, we declare the slots in the header file
```javascript```
private slots:
      void ClearSlot();
      
```
     Then we implement the functions in the cpp file 
      
   ```javascript```
void toDoApp::ClearSlot()
{
    ui->Pending->clear();
    ui->pesistent->clear();
    ui->completed->clear();

    QFile file("/Users/hp/Desktop/save.txt");
    if (file.open(QIODevice::ReadWrite| QIODevice::Text)){
        file.resize(0);

    }
}
```





