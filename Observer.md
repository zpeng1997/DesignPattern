### Obserever design pattern
- 通知依赖关系
    - 一个对象的状态发生改变, 所有依赖对象(观察者对象)都会被通知
    - 使用面向对象技术, 实现关系的松耦合。

### 错误例子 进度条的实现
```C++
// MainFrame.cpp
class MainFrame : public Frame
{
	TextBox* txtFilePath;
	TextBox* txtFileNumber;
	ProgressBar* progressBar; // ProgressBar的实现方法会不断的改变(因为需求), 

public:
	void Button1_Click(){

		string filePath = txtFilePath->getText();
		int number = atoi(txtFileNumber->getText().c_str());

		FileSplitter splitter(filePath, number, progressBar);

		splitter.split();

	}
};

// FileSplitter.cpp
class FileSplitter
{
	string m_filePath;
	int m_fileNumber;
	ProgressBar* m_progressBar;

public:
	FileSplitter(const string& filePath, int fileNumber, ProgressBar* progressBar) :
		m_filePath(filePath), 
		m_fileNumber(fileNumber),
		m_progressBar(progressBar){
	}

	void split(){
		//1.读取大文件
		//2.分批次向小文件中写入
		for (int i = 0; i < m_fileNumber; i++){
			//...
			float progressValue = m_fileNumber;
			progressValue = (i + 1) / progressValue;
			m_progressBar->setValue(progressValue);
		}

	}
};
```
- 因为ipressBar是变化点, 因为需求的改变导致它的实现会不断的更改, 这样违背DIP原则

### 解决方案, 抽象, 封装变化点
- 多继承的用法, 一个继承主要的类, 其它的都继承接口
```C++
// FileSplitter.cpp
class IProgress{
public:
	virtual void DoProgress(float value)=0;
	virtual ~IProgress(){}
};

class FileSplitter
{
	string m_filePath;
	int m_fileNumber;

    // IProgress iprogress; 方法已经实现了 接口倒置原则, 下面扩充只是允许多个观察者(这里的操作只支持MainFrame一个观察者)
	List<IProgress*>  m_iprogressList; // 抽象通知机制，支持多个观察者
	
public:
	FileSplitter(const string& filePath, int fileNumber) :
		m_filePath(filePath), 
		m_fileNumber(fileNumber){
	}

	void split(){
		//1.读取大文件

		//2.分批次向小文件中写入
		for (int i = 0; i < m_fileNumber; i++){
			//...

			float progressValue = m_fileNumber;
			progressValue = (i + 1) / progressValue;
			onProgress(progressValue);//发送通知
		}

	}

    // 也可以进一步抽象, 把addIProgress RemoveIProgress OnProgress放到基类当中, FileSplitter去继承它
	void addIProgress(IProgress* iprogress){
		m_iprogressList.push_back(iprogress);
	}

	void removeIProgress(IProgress* iprogress){
		m_iprogressList.remove(iprogress);
	}
protected:
	virtual void onProgress(float value){ // 设置虚函数就是允许, 子类更改默认的onProgress方法
		
		List<IProgress*>::iterator itor = m_iprogressList.begin();

		while (itor != m_iprogressList.end() )
			(*itor)->DoProgress(value); //更新进度条
			itor++;
		}
	}
};

// MainFrame.cpp
class MainForm : public Form, public IProgress // 多继承, 一个基本类, 其它的都继承接口
{
	TextBox* txtFilePath;
	TextBox* txtFileNumber;

	ProgressBar* progressBar;

public:
	void Button_Click(){

		string filePath = txtFilePath->getText();
		int number = atoi(txtFileNumber->getText().c_str());

		ConsoleNotifier cn;

		FileSplitter splitter(filePath, number);

		splitter.addIProgress(this); //订阅通知
		splitter.addIProgress(&cn)； //订阅通知

		splitter.split();

		splitter.removeIProgress(this);

	}

	virtual void DoProgress(float value){
		progressBar->setValue(value);
	}
};

class ConsoleNotifier : public IProgress { // 推荐这种方法, 单一职责, 实现实现层的松耦合
public:
	virtual void DoProgress(float value){
		cout << ".";
	}
};
```

### 总结
- Observer 可以使我们独立的改变 目标 和 观察者, 使得两者之间的依赖关系达到松耦合
- 目标发送通知时, 无需指定观察者onProgress(progressValue), 不需要指定ProgressBar对象, 通知自动传播(根据onProgress中指定的方法)
- 观察者自己决定是否需要订阅通知, 而目标对象一无所知
- Observer模式是基于UI框架的一个非常常用的设计模式, 也是MVC模式中的一个基本模式
- 类图:
    - ![avatar](https://github.com/zpeng1997/DesignPattern/blob/master/Observer.png)