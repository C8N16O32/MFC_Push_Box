/*stdafx.h的内容：
#pragma once

#include "targetver.h"

#include "afxwin.h"
*/
/***************************************************************************  推箱子  *************************************************************************************/

#include "stdafx.h"

//mfc程序架构/绘图架构

//程序框架
class CMyFrameWnd : public CFrameWnd {
	DECLARE_MESSAGE_MAP()
public:
	int OnCreate(LPCREATESTRUCT p);
	void OnMouseMove(UINT nKey, CPoint pt);
	void OnLButtonUp(UINT nKey, CPoint pt);
	void OnLButtonDown(UINT nKey, CPoint pt);
	void OnRButtonUp(UINT nKey, CPoint pt);
	void OnRButtonDown(UINT nKey, CPoint pt);
	void OnTimer(UINT_PTR ptr);
	void OnSize(UINT uint, int x, int y);
	void OnPaint();
	void doublebuffer(CDC *pDC);
	UINT nKey; CPoint pt;
	CSize screensize = { 1280,720 };
	time_t timems, timemsold;
};

//消息反射宏表
BEGIN_MESSAGE_MAP(CMyFrameWnd, CFrameWnd)
	ON_WM_CREATE()
	ON_WM_MOUSEMOVE()
	ON_WM_LBUTTONDOWN()
	ON_WM_LBUTTONUP()
	ON_WM_RBUTTONDOWN()
	ON_WM_RBUTTONUP()
	ON_WM_TIMER()
	ON_WM_SIZE()
	ON_WM_PAINT()
END_MESSAGE_MAP()

//窗口类
class CMyWinApp : public CWinApp {
public:
	CMyWinApp() {};
	virtual BOOL InitInstance() {
		CMyFrameWnd* pFrame = new CMyFrameWnd;
		pFrame->Create(NULL, (LPCTSTR)_T("推箱子"));
		m_pMainWnd = pFrame;
		pFrame->ShowWindow(SW_SHOW);
		pFrame->UpdateWindow();
		return TRUE;
	}
};
//窗口对象创建
CMyWinApp theApp;

/*****************************************************************           宏           *****************************************************************/

//箱子个数+人的个数 3+1 暂时是定值
#define N 4

/*****************************************************************        数据结构        *****************************************************************/

//循环数组实现的队列/栈两用结构
template<class T>
class QUEUE {
	//调整幅度
	int m_dc = 16, m_shc = 0, m_dsh = 8;
public:
	//数据
	T*v = NULL; int m_head = 0, m_end = 0, m_capacity = 0;
	//信息
	bool isempty() { if (m_head == m_end)return 1; return 0; };
	int size() { return m_end - m_head; };
	void setdcstepshstep(int dc, int dsh) { m_dc = dc; m_dsh = dsh; return; };
	//调整大小
	void changesize(int newcapacity) {
		int s1 = m_head, l1 = size(), s2 = 0, l2 = 0;
		if (m_end > m_capacity) { l2 = m_end - m_capacity; l1 -= l2; };
		//printf("changesize:%d[%d,%d) to %d[%d,%d) s1 %d l1 %d s2 %d l2 %d\n", m_capacity, m_head, m_end, newcapacity, 0, l1 + l2, s1, l1, s2, l2);
		T*temp = new T[newcapacity]; int i;
		for (i = s1; i < s1 + l1; i++)temp[i - s1] = v[i];
		for (i = s2; i < s2 + l2; i++)temp[i - s2 + l1] = v[i];
		delete[]v; v = temp; temp = NULL;
		m_head = 0; m_end = l1 + l2; m_capacity = newcapacity;
		return;
	};
	void checkcapa(int ns/*需要的额外空间数*/) {
		int sizeold = QUEUE::size() + ns, size = m_dc;
		while (sizeold > size)size *= 2;
		if (size > m_capacity) { changesize(size); m_shc = m_dsh; return; };
		if (m_shc) { m_shc--; return; };
		if (size < m_capacity)changesize(size);
		return;
	};
	//入队/出队
	void push_end(T&inp) {
		checkcapa(1);
		v[m_end%m_capacity] = inp; m_end++;
		return;
	};
	void push_end_notref(T inp) {
		checkcapa(1);
		v[m_end%m_capacity] = inp; m_end++;
		return;
	};
	T&head() {
		return v[m_head];
	};
	void pop_head() {
		m_head++;
		if (m_head == m_capacity) {
			m_head -= m_capacity; m_end -= m_capacity;
		};
		checkcapa(0);
		return;
	};
	T&rhead() {
		return v[(m_end - 1) % m_capacity];
	};
	void pop_end() {
		m_end--;
		checkcapa(0);
		return;
	};
	//清空
	void reset() { delete[]v; v = NULL; m_head = 0; m_end = 0; m_capacity = 0; }
	//构造/析构
	virtual ~QUEUE<T>() {
		delete[]v; v = NULL;
	};
};


//点或向量
struct P {
	int i, j;
	bool operator==(struct P &y) const { if (i == y.i&&j == y.j)return 1; return 0; };
	P operator+(struct P &y) const { return{ i + y.i,j + y.j }; };
	void operator+=(struct P &y) { i += y.i; j += y.j; };
	void operator-=(struct P &y) { i -= y.i; j -= y.j; };
};
//可动元素集合
struct DATA {
	P p[N];
	int count;
	void print(char **v0, int m, int n) {
		char v[8][8] = {};
		int i, j;
		for (i = 0; i < m; i++)for (j = 0; j < n; j++)v[i][j] = v0[i][j];
		for (i = 1; i < 4; i++)if (v[p[i].i][p[i].j] != '@')v[p[i].i][p[i].j] = '*'; else v[p[i].i][p[i].j] = '&';
		v[p[0].i][p[0].j] = 'X';
		for (i = 0; i < m; i++)printf("%.*s\n", n, v[i]);
		printf("%d\n", count);
	}
};
//地图
struct MAP {
	//地图
	int m, n;
	int pushlimit = 1;
	char **v = NULL;//地面. 墙壁# 终点@
	DATA start;//初态

	//修改地图尺寸
	void changesize(int im, int in) {
		char **vtemp = new char*[im];
		for (int i = 0; i < im; i++) {
			vtemp[i] = new char[in];
			for (int j = 0; j < in; j++)vtemp[i][j] = '.';
		}
		int m2 = m < im ? m : im, n2 = n < in ? n : in;
		for (int i = 0; i < m2; i++)
			for (int j = 0; j < n2; j++)vtemp[i][j] = v[i][j];
		for (int i = 0; i < m; i++)delete[]v[i];
		delete[]v; v = vtemp; vtemp = NULL;
		m = im, n = in;
	}
	//赋值 m n pushlimit v start
	void operator=(MAP&y) {
		//复制参数
		pushlimit = y.pushlimit;
		start = y.start;
		//修改地图尺寸
		if (m != y.m || n != y.n) {
			changesize(y.m, y.n);
		}
		//复制地图
		for (int i = 0; i < m; i++)
			for (int j = 0; j < n; j++)
				v[i][j] = y.v[i][j];
	}
	//清空地图 pushlimit v start
	void reset() {
		static DATA d0 = { 0 };
		start = d0;
		pushlimit = 1;
		for (int i = 0; i < m; i++)
			for (int j = 0; j < n; j++)
				v[i][j] = '.';
	};

	//人物移动
	P caloff(int ord) { static const int offi[] = { -1,0,1,0 }, offj[] = { 0,1,0,-1 }; return{ offi[ord],offj[ord] }; };
	DATA move(DATA&cur, P &off, int &flag) {
		DATA temp = cur;
		flag = 0;
		P cp = temp.p[0];
		int c = 0, f, pushcount = 0;
		while (1) {
			cp += off;
			f = -1; for (int i = 0; i < N; i++)if (temp.p[i] == cp)f = i;
			temp.p[c] = cp;
			if (f == -1) break;
			else c = f;
			pushcount++;
		};
		if (pushcount > pushlimit)return temp;

		P *ccp = NULL; for (int i = 0; i < N; i++) {
			ccp = temp.p + i;
			if (ccp->i < 0 || ccp->i >= m || ccp->j < 0 || ccp->j >= n)return temp;
			if (v[ccp->i][ccp->j] == '#')return temp;
		};
		flag = 1;
		temp.count++;
		return temp;
	}

	//通关判定
	bool isend(DATA&cur) { for (int i = 1; i < 4; i++)if (v[cur.p[i].i][cur.p[i].j] != '@')return 0; return 1; };

	//构造/析构 mn v
	explicit MAP(int im, int in) {
		//printf("开始构造\n");
		m = im, n = in;
		v = new char*[m];
		for (int i = 0; i < m; i++)v[i] = new char[n];
		reset();
	};
	~MAP() {
		//printf("开始析构\n");
		for (int i = 0; i < m; i++)delete[]v[i];
		delete[]v; v = NULL;
	};

};

/*****************************************************************        对象声明        *****************************************************************/


//*********************************************  内核  *********************************************

struct PLAY {
	//地图
	MAP &map;//构造函数中完成初始化

	//游玩记录
	QUEUE<DATA> qplay;

	//获取当前状态
	DATA getcur();
	//检查是否通关
	bool isend_play();

	//推导内容(在光标处于以玩家位置为中心的十字线上 会尝试让角色走到光标下方 生成一个推导内容 当写入记录时写入此推导内容)
	P lastpos;
	bool lastflag;
	DATA lastresult;
	//更新推导内容
	bool Set(P pos);
	//推导内容写入记录
	void Do(P pos);
	//记录的清零与撤回
	void Reset();
	void Undo();

	//构造/析构
	PLAY(MAP &imap);
	void leave();

	//自动解算部分
	int sence = 0;//0默认 1为自动解算

	//搜索结果
	int bfslastflag = 2; QUEUE<DATA> bfslastresult;

	struct BFSINFO {
		//搜索过程
		int n1, n2, n3, nmax;
		void reset_1(MAP&map);
		void update_1(int plus, int qlen, int curcount);
		void reset_2();
		void update_2(int plus, int qlen, int curcount);
	}bfsinfo;

	//搜索
	struct SOLVE {
		//地图
		MAP &map;
		//运算结果
		int count[8][8][8][8][8][8][8][8] = {};
		bool tag[8][8][8][8][8][8][8][8] = {};
		DATA start, end;
		//bfs
		int bfsmain(DATA start, int &issolve, BFSINFO&bfsinfo);
		//寻路
		void pathmain(QUEUE<DATA> &qre, BFSINFO&bfsinfo);
		//构造/析构
		explicit SOLVE(MAP &imap);
	};

	void solvemain();
};

struct EDIT {
	MAP &map, temp;
	int toolflag = 0;//默认0为空地
	char toollist[5] = { '.','#','@','X','*' };

	//左标签相关
	void Selecttool(int New);
	//棋盘
	void Do(P pos, int isleftclick = 1);
	//右标签相关
	void Setsize(int im, int in);
	void Setpushlimit(int il);
	//设计检查 其中len为48及以上
	bool Check(char *text, int len);
	void scanmap();
	void printmap();
	//构造/析构
	EDIT(MAP &imap);
};


//*********************************************  图形  *********************************************


//*********************************************  框架  *********************************************

//模板 函数LD：将闭包lambda表达式降级为函数指针
template<typename T>struct LambdaDecay_impl;
template<typename C, typename R, typename...Args>
struct LambdaDecay_impl<R(C::*)(Args...) const> { using type = R(*)(Args...); };
template<typename T>struct LambdaDecay { using type = typename LambdaDecay_impl<decltype(&T::operator())>::type; };
template<typename T>using LDR = typename LambdaDecay<T>::type;
template<typename F>LDR<F> LD(F lambda) { return LDR<F>(lambda); };

//模板：函数形参
template<typename R, typename ...T>
struct S1 {
	//基础
	R(*F)(T...);
	S1<R, T...>() { F = NULL; };
	S1<R, T...>(decltype(F) f0) { F = f0; };
	R operator()(T ...args) { return F(args...); };
};
template<typename R, typename ...T>
S1<R, T...> HS1(R(*F)(T...)) { return S1<R, T...>(F); };//一般函数


//游戏框架
#define args_core CPoint cp, int clicktype, PLAY*play, EDIT*edit, CRect &rect
#define args_plot CDC*pDC, PLAY*play, EDIT*edit, CRect &rect, int ptype, CPen &pen, CPen &penmark
auto GAME_lambdasample_core = [](args_core) ->void {};
auto GAME_lambdasample_plot = [](args_plot) ->void {};
class GAME {
public:
	MAP map;
	//界面
	EDIT edit;
	PLAY play;
	int sence = 0;
	//按钮队列
	using F_core_type = decltype(HS1(LD(GAME_lambdasample_core)));
	using F_plot_type = decltype(HS1(LD(GAME_lambdasample_plot)));
	template<class F_core, class F_plot>struct button { F_core fc; F_plot fp; int ptype; CRect rect0, rect; CSize ss; };
	using b_type = button<F_core_type, F_plot_type>;
	using Q_type = QUEUE<b_type>;
	Q_type editbuttonqueue, playbuttonqueue;
	void buttonqueue_setfun();
	//自适应缩放
	struct SCALEDATA {
		int mul = 1, div = 1, offx, offy;
		void set(CSize &screensize);
	}scaledata;
	CPen pen, penmark;
	void scalemain(CSize screensize);
	//处理输入
	int nkeyold = 0;
	void inputmain(CPoint cp, int nKey);;
	//处理绘图
	void plotmain(CDC*pDC);;

	//构造/析构
	GAME();;

}game;


/*****************************************************************          内核          *****************************************************************/


//*********************************************  PLAY  *********************************************

//获取当前状态

inline DATA PLAY::getcur() {
	if (qplay.isempty())return map.start;
	return qplay.rhead();
}

//检查是否通关

inline bool PLAY::isend_play() { return map.isend(getcur()); }

//输入准备

inline bool PLAY::Set(P pos) {
	//1 不与上一次计算重合
	if (pos == lastpos)return 0;
	lastpos = pos;
	lastflag = 0;
	// 2 不与当前点重合
	DATA cur = getcur();
	if (pos == cur.p[0]) return 0;
	// 3 处于十字线上
	int f = 0, oi = pos.i - cur.p[0].i, oj = pos.j - cur.p[0].j;
	if (oi)f++;
	if (oj)f++;
	if (f != 1)return 0;
	// 4 尝试递推动作
	int len = oi + oj; if (len < 0)len = -len;
	if (oi>0)oi = 1; if (oi<0)oi = -1;
	if (oj>0)oj = 1; if (oj<0)oj = -1;
	P off = { oi,oj };
	DATA temp = cur;
	for (int i = 0; i < len; i++) {
		temp = map.move(temp, off, f);
		if (f == 0)return 0;
	}
	lastflag = 1;
	lastresult = temp;
	return 1;
}

//输入

inline void PLAY::Do(P pos) {
	if (!(pos == lastpos))Set(pos);
	if (!lastflag)return;
	qplay.push_end(lastresult);
}

//清零与撤回

inline void PLAY::Reset() { while (!qplay.isempty())qplay.pop_end(); }

inline void PLAY::Undo() { if (!qplay.isempty())qplay.pop_end(); }

//构造/析构

inline PLAY::PLAY(MAP & imap) :map(imap) {}

inline void PLAY::leave() { qplay.reset(); lastflag = 0; bfslastflag = 2; bfslastresult.reset(); }

inline void PLAY::solvemain() {
	sence = 1;
	SOLVE solve(map);
	int flag = 1;
	solve.bfsmain(getcur(), flag, bfsinfo);
	bfslastflag = flag;
	bfslastresult.reset();
	if (bfslastflag)solve.pathmain(bfslastresult, bfsinfo);
	sence = 0;
	/*测试用 将自动记录覆盖到手动记录*/
	while (!bfslastresult.isempty()) {
		qplay.push_end(bfslastresult.rhead());
		bfslastresult.pop_end();
	}
}

//PLAY::BFSINFO

inline void PLAY::BFSINFO::reset_1(MAP & map) {
	n1 = 0; n2 = 0; n3 = 0; nmax = map.m*map.n;
	nmax = nmax*nmax*nmax*nmax; n3 = nmax;
}

inline void PLAY::BFSINFO::update_1(int plus, int qlen, int curcount) { n2 += plus; n1 = n2 - qlen; }

inline void PLAY::BFSINFO::reset_2() { n3 = n1; n1 = 0; n2 = 0; }

inline void PLAY::BFSINFO::update_2(int plus, int qlen, int curcount) { n2 += plus; n1 = n2 - qlen; }

//PLAY::SOLVE

//在搜索过程中使用 根据数据绘制当前界面 包含当前尝试情况与一个进度条
auto pspecial_bfsinfo = [&](DATA&cur) {
	//设置到记录
	game.play.lastflag = 1;
	game.play.lastresult = cur;

	//定义对象
	CDC MemDC, *pDC = theApp.m_pMainWnd->GetDC(); CBitmap MemBitmap;
	//使用函数初始化1：建立与屏幕显示兼容的内存显示设备
	MemDC.CreateCompatibleDC(NULL);
	//使用函数初始化2：建立与屏幕显示兼容的位图
	CMyFrameWnd* pmf = (CMyFrameWnd*)(theApp.m_pMainWnd);
	MemBitmap.CreateCompatibleBitmap(pDC, pmf->screensize.cx, pmf->screensize.cy);
	//将位图与内存显示设备进行链接
	CBitmap *pOldBit = MemDC.SelectObject(&MemBitmap);
	//调用功能：用背景色将位图清除干净 TODO: 设置清屏颜色
	MemDC.FillSolidRect(0, 0, pmf->screensize.cx, pmf->screensize.cy, RGB(196, 196, 240));

	game.scalemain(pmf->screensize);
	game.plotmain(&MemDC);

	//位图复制：从显示设备复制到主显示设备PDC 此时刷新画面
	pDC->BitBlt(0, 0, pmf->screensize.cx, pmf->screensize.cy, &MemDC, 0, 0, SRCCOPY);
	//使用函数清理：清理两个对象
	MemBitmap.DeleteObject();
	MemDC.DeleteDC();
};

inline int PLAY::SOLVE::bfsmain(DATA start, int & issolve, BFSINFO & bfsinfo) {
	this->start = start;
	//标记空间
	auto checktag = [&](DATA &cur) ->bool {return tag[cur.p[0].i][cur.p[0].j][cur.p[1].i][cur.p[1].j][cur.p[2].i][cur.p[2].j][cur.p[3].i][cur.p[3].j]; };
	auto marktag = [&](DATA &cur) {tag[cur.p[0].i][cur.p[0].j][cur.p[1].i][cur.p[1].j][cur.p[2].i][cur.p[2].j][cur.p[3].i][cur.p[3].j] = 1; };
	auto markcount = [&](DATA &cur) {count[cur.p[0].i][cur.p[0].j][cur.p[1].i][cur.p[1].j][cur.p[2].i][cur.p[2].j][cur.p[3].i][cur.p[3].j] = cur.count; };

	//队列空间
	QUEUE<DATA> q;

	//中间值
	bfsinfo.reset_1(map);

	//bfs求解
	issolve = 0;
	DATA cur = start, temp; int flag;
	marktag(cur); markcount(cur);
	q.push_end(cur);
	while (!q.isempty()) {
		cur = q.head(); q.pop_head();
		if (map.isend(cur)) {
			issolve = 1; end = cur;
			pspecial_bfsinfo(end);
			return cur.count;
		}
		for (int i = 0; i < 4; i++) {
			temp = map.move(cur, map.caloff(i), flag);
			if (flag && (!checktag(temp))) {
				marktag(temp); markcount(temp);
				q.push_end(temp);

				//中间值
				bfsinfo.update_1(1, q.size(), temp.count);
				if (bfsinfo.n2 % 10000 == 0) {

					pspecial_bfsinfo(temp);

				}
			}
		}
	}
	return -1;
}

inline void PLAY::SOLVE::pathmain(QUEUE<DATA>& qre, BFSINFO & bfsinfo) {
	//标记空间
	auto checktag = [&](DATA &cur) ->bool {return tag[cur.p[0].i][cur.p[0].j][cur.p[1].i][cur.p[1].j][cur.p[2].i][cur.p[2].j][cur.p[3].i][cur.p[3].j]; };
	auto getcount = [&](DATA &cur) ->int {return count[cur.p[0].i][cur.p[0].j][cur.p[1].i][cur.p[1].j][cur.p[2].i][cur.p[2].j][cur.p[3].i][cur.p[3].j]; };

	//栈空间
	QUEUE<DATA> qtemp;

	//中间值
	bfsinfo.reset_2();

	//寻路
	auto findpre = [&](DATA temp, P &off) {
		int c = 0;
		P cp = temp.p[c] + off;
		//不能超界 不能是墙 不能是另一个箱子/人 不满足移动条件则直接退出
		int m = map.m, n = map.n; char**v = map.v;
		if (cp.i < 0 || cp.i >= m || cp.j < 0 || cp.j >= n)return;
		if (v[cp.i][cp.j] == '#')return;
		for (int i = 0; i < N; i++)if (i != c&&cp == temp.p[i])return;
		//满足移动条件则移动 然后按后继物体不变的情况入队（允许重复）
		temp.p[c] = cp; if (checktag(temp))qtemp.push_end(temp);
		//寻找后继物体 若有则一定能跟随 然后入队 直到找不到后继物体
		cp -= off;
		while (1) {
			cp -= off;
			c = -1; for (int i = 0; i < N; i++)if (cp == temp.p[i])c = i;
			if (c == -1)return;
			temp.p[c] += off; if (checktag(temp)) {
				qtemp.push_end(temp);

				//中间值
				bfsinfo.update_2(1, qtemp.size(), temp.count);
				if (bfsinfo.n2 % 100000 == 0) {
					//bfs搜索之后的寻路速度很快 不需要打印
				}
			}
		};
	};

	//求解
	DATA cur = end, temp; int ccount, iss;
	int offi[] = { -1,0,1,0 }, offj[] = { 0,1,0,-1 }; P off;
	while (getcount(cur) != start.count) {
		qre.push_end(cur);
		for (int i = 0; i < 4; i++) {
			off = { offi[i],offj[i] };
			findpre(cur, off);
		}
		ccount = getcount(cur);
		iss = 0;
		while (!qtemp.isempty()) {
			temp = qtemp.head(); qtemp.pop_head();
			if (getcount(temp) == ccount - 1) {
				iss = 1; cur = temp; cur.count = ccount - 1; break;
			}
		}
		while (!qtemp.isempty()) qtemp.pop_head();
		if (iss == 0) {
			printf("反推过程时无法继续\n");
			scanf("%d", &iss);
		}
	}

}

inline PLAY::SOLVE::SOLVE(MAP & imap) :map(imap) {}


//*********************************************  EDIT  *********************************************

//左标签

inline void EDIT::Selecttool(int New) { toolflag = New; }

inline void EDIT::Do(P pos, int isleftclick) {
	if (toolflag < 3)temp.v[pos.i][pos.j] = isleftclick ? toollist[toolflag] : toollist[0];
	else if (toolflag == 3)temp.start.p[0] = pos;
	else if (toolflag == 4 && !(temp.start.p[N - 1] == pos)) {
		for (int i = 1; i < N - 1; i++)temp.start.p[i] = temp.start.p[i + 1];
		temp.start.p[N - 1] = pos;
	};
}

//右标签

inline void EDIT::Setsize(int im, int in) {
	temp.changesize(im, in);
}

inline void EDIT::Setpushlimit(int il) { temp.pushlimit = il; }

//设计检查 len建议48

inline bool EDIT::Check(char * text, int len) {
	int i, j, m = temp.m, n = temp.n;
	for (i = 0; i < len; i++)text[i] = 0;
	int c = 0; for (i = 0; i < m; i++)for (j = 0; j < n; j++)if (temp.v[i][j] == '@')c++;
	if (c != N - 1) { sprintf(text, "修改设计：地图应该拥有%d个检查点", N - 1); return 0; }
	c = 1; for (i = 0; c&&i < N; i++)for (j = 0; c&&j < N; j++)if (c&&j != i&&temp.start.p[i] == temp.start.p[j])c = 0;
	if (c == 0) { sprintf(text, "修改设计：箱子/起点相互之间不能重叠", N - 1); return 0; }
	c = 1; for (i = 0; c&&i < N; i++)if (temp.start.p[i].i < 0 || temp.start.p[i].i >= m || temp.start.p[i].j < 0 || temp.start.p[i].j >= n)c = 0;
	if (c == 0) { sprintf(text, "修改设计：箱子/起点不能放在地图外", N - 1); return 0; }
	c = 1; for (i = 0; c&&i < N; i++)if (temp.v[temp.start.p[i].i][temp.start.p[i].j] == '#')c = 0;
	if (c == 0) { sprintf(text, "修改设计：箱子/起点不能放在墙壁上", N - 1); return 0; }
	sprintf(text, "*检查通过*", N - 1); return 1;
}

inline void EDIT::scanmap() { temp = map; }

inline void EDIT::printmap() { map = temp; }

inline EDIT::EDIT(MAP & imap) :map(imap), temp(8, 8) {}


/*****************************************************************          图形          *****************************************************************/

//文本
void ptext(CDC*pDC, char*text, CRect rect, COLORREF textrgb = RGB(0, 0, 0)) {
	CFont cfdb; cfdb.CreatePointFont(rect.Height() * 7, (LPCTSTR)_T("黑体"));
	pDC->SelectObject(cfdb); pDC->SetTextColor(textrgb);
	pDC->TextOut(rect.CenterPoint().x - pDC->GetTextExtent(text).cx / 2
		, rect.CenterPoint().y - pDC->GetTextExtent(text).cy / 2, text);
	cfdb.DeleteObject();
	return;
}

//填充方形
void prect(CDC*pDC, CRect rect, COLORREF color = RGB(128, 128, 128)) {
	pDC->FillSolidRect(rect, color);
}

//画笔
void setpen(CPen *penmark, CPen *pen,int width) {
	int wm = width / 80 + 2, wd = (wm - 2) / 3;
	penmark->CreatePen(PS_SOLID, wm, RGB(0, 0, 0)); pen->CreatePen(PS_SOLID, wd, RGB(0, 0, 0));
}

//线_圆形
void pcircle_setrect(CRect &rect, CPoint pc, double r) { rect.SetRect((int)(pc.x - r + 0.5), (int)(pc.y - r + 0.5), (int)(pc.x + r + 0.5), (int)(pc.y + r + 0.5)); };
void pcircle(CDC*pDC, CPen *pen, CRect r0, double shrinkrate = 1) {
	CPen *oldPen = pDC->SelectObject(pen);
	int dl = r0.Width() / 2 * (1 - shrinkrate) + 0.5;
	r0.DeflateRect(dl, dl, dl, dl);
	pDC->Ellipse(&r0);
	pDC->SelectObject(oldPen);
};

//线_直线
void pline(CDC*pDC, CPen *pen, CPoint p1, CPoint p2) {
	CPen *oldPen = pDC->SelectObject(pen);
	pDC->MoveTo(p1); pDC->LineTo(p2);
	pDC->SelectObject(oldPen);
};


//输入点转化为输入坐标
P pchess_calij(CPoint cp, CRect rect, MAP&map){
	int m = map.m, n = map.n, i, j;
	i = (cp.y - rect.top) / (rect.Height() / m); if (i >= m)i = m - 1;
	j = (cp.x - rect.left) / (rect.Width() / n); if (j >= n)j = n - 1;
	return{ i,j };
};

//绘制中心棋盘
void pchess(CDC*pDC, MAP&map, DATA&cur, CRect rect) {
	COLORREF dark[4] = { RGB(46,117,181),RGB(30,78,121),RGB(47,84,150),RGB(31,56,100) }
		, light[4] = { RGB(222,235,246), RGB(189,215,238),RGB(217,226,243),RGB(180,198,231) }
	, player = RGB(63, 63, 63), box = RGB(191, 144, 0), hole = RGB(168, 208, 141);
	int choose = 0;
	int m = map.m, n = map.n;
	int w0 = rect.left, w = rect.Width(), h0 = rect.top, h = rect.Height();

	CRect temp;
	auto set = [&](int i, int j, int m, int n, double der = 0) {temp.SetRect(w*j / n, h*i / m, w*(j + 1) / n, h*(i + 1) / m); temp.OffsetRect(w0, h0); int dx = der*w/n, dy = der*h/m; temp.DeflateRect(dx, dy); };
	for (int i = 0; i < m; i++)
		for (int j = 0; j < n; j++) {
			set(i, j, m, n);
			switch (map.v[i][j]) {
			case '.':prect(pDC, temp, light[(i + j) % 4]); break;
			case '#':prect(pDC, temp, dark[(m - 1 - i + j) % 4]); break;
			case '@':prect(pDC, temp, hole); break;
			}
		}
	double deflaterate = 0.1;
	set(cur.p[0].i, cur.p[0].j, m, n, deflaterate);
	prect(pDC, temp, player);
	for (int i = 1; i < N; i++) {
		set(cur.p[i].i, cur.p[i].j, m, n, deflaterate);
		prect(pDC, temp, box);
	}
};

//绘制进度条
void pcalbar(CDC*pDC, PLAY::BFSINFO &info, CRect rect) {
	//进度条分界点
	int s1 = info.n1, s2 = info.n2, s3 = info.n3, smax = info.nmax;
	double d1 = s1*1.0 / smax, d2 = s2*1.0 / smax, d3 = s3*1.0 / smax;
	//计算坐标
	smax = rect.right;
	int x0 = rect.left, x1 = (smax - x0)*d1 + x0, x2 = (smax - x0)*d2 + x0, x3 = (smax - x0)*d3 + x0;
	CRect r1 = rect, r2 = rect, r3 = rect; r1.right = x1; r2.right = x2; r3.right = x3;
	//进度条颜色
	COLORREF c1 = RGB(63, 209, 178), c2 = RGB(8, 131, 216), c3 = RGB(14, 82, 152);
	prect(pDC, r3, c3); prect(pDC, r2, c2); prect(pDC, r1, c1);
}


/*****************************************************************          框架          *****************************************************************/


//*********************************************  GAME  *********************************************

//GAME::SCALEDATA

inline void GAME::SCALEDATA::set(CSize & screensize) {
	CRect def = { 0,0,1280,960 }, ss = { 0,0,screensize.cx,screensize.cy };
	int cx0 = def.Width(), cy0 = def.Height();
	double limfactor = 0.3; int xm = cx0*limfactor, ym = cy0*limfactor;
	if (ss.Width() < xm || ss.Height() < ym) ss = { 0,0,xm,ym }, mul = xm, div = cy0;
	else {
		mul = ss.Width(); div = cx0;
		if (mul*1.0 / div > ss.Height()*1.0 / cy0)mul = ss.Height(), div = cy0;
	}
	offx = (ss.Width() - cx0*mul / div) / 2; offy = 0;
}

//自适应缩放、输入检测与响应、绘图

inline void GAME::scalemain(CSize screensize) {
	scaledata.set(screensize);
	auto qfun = [&](Q_type &q)->void {
		if (q.isempty())return;
		int capa = q.m_capacity; b_type *b = NULL;
		for (int i = q.m_head; i < q.m_end; i++) {
			b = q.v + i%capa;
			if (b->ss != screensize) {
				b->ss = screensize;
				b->rect = b->rect0.MulDiv(scaledata.mul, scaledata.div);
				b->rect.OffsetRect(scaledata.offx, scaledata.offy);
			}
		}
	};
	switch (sence) {
	case 0:qfun(editbuttonqueue); break;
	case 1:qfun(playbuttonqueue); break;
	default:break;
	}
}

inline void GAME::inputmain(CPoint cp, int nKey) {
	//clicktype
	int clicktype = 0;
	if ((nkeyold & MK_LBUTTON) == 0 && (nKey & MK_LBUTTON) != 0)
		clicktype = 1;
	if ((nkeyold & MK_RBUTTON) == 0 && (nKey & MK_RBUTTON) != 0)
		clicktype = 2;

	auto qfun = [&](Q_type &q)->void {
		if (q.isempty())return;
		int capa = q.m_capacity;
		for (int i = q.m_head; i < q.m_end; i++)
			q.v[i%capa].fc(cp, clicktype, &play, &edit, q.v[i%capa].rect);
	};
	switch (sence) {
	case 0:qfun(editbuttonqueue); break;
	case 1:if (game.play.sence == 0)qfun(playbuttonqueue); break;
	default:break;
	}
	nkeyold = nKey;
}

inline void GAME::plotmain(CDC * pDC) {

	auto qfun = [&](Q_type &q)->void {
		if (q.isempty())return;
		int capa = q.m_capacity;
		for (int i = q.m_head; i < q.m_end; i++)
			q.v[i%capa].fp(pDC, &play, &edit, q.v[i%capa].rect, q.v[i%capa].ptype, pen, penmark);
	};
	switch (sence) {
	case 0:qfun(editbuttonqueue); break;
	case 1:qfun(playbuttonqueue); break;
	default:break;
	}
}

//构造/析构

inline GAME::GAME() :map(8, 8), edit(map), play(map) {
	buttonqueue_setfun();
	setpen(&penmark, &pen, 640);
}


/*****************************************************************      按钮队列内容      *****************************************************************/

void GAME::buttonqueue_setfun(){

#define begin_edit editbuttonqueue.push_end_notref({ HS1(LD( [](args_core)->void {
#define begin_play playbuttonqueue.push_end_notref({ HS1(LD( [](args_core)->void {
#define plot })),HS1(LD( [](args_plot)->void {
#define setrect0 })) ,0,{
#define end },{0,0,1,1},{1,1}});

//颜色表
#define 橙色暗 RGB(90,40,8)
#define 橙色 RGB(183,71,42)
	/************************  编辑界面-中心  ************************/

	//背景
	begin_edit plot
		switch (ptype) {
		case 0:
			prect(pDC, rect, RGB(51, 196, 129)); break;
		default:break;
		}
	setrect0 
		0, 0, 1280, 960 
	end

	//中央棋盘
	begin_edit
		if (!rect.PtInRect(cp))return;
		P p = pchess_calij(cp, rect, play->map);
		switch (clicktype) {
		case 1:edit->Do(p, 1); break;
		case 2:edit->Do(p, 0); break;
		default:break;
		}
	plot
		switch (ptype) {
		case 0:pchess(pDC, edit->temp, edit->temp.start, rect); break;
		default:break;
		}
	setrect0
		200, 200, 1080, 760
	end

	/************************  编辑界面-左侧  ************************/
#define selectbar(num,text) \
		begin_edit \
			if (!rect.PtInRect(cp))return; \
		switch (clicktype) { \
		case 1:edit->Selecttool(num); break; \
		default:break; \
		} \
		plot \
			switch (ptype) { \
			case 0: \
				prect(pDC, rect, 橙色); \
				ptext(pDC, text, rect, 橙色暗); break; \
			default:break; \
			} \
		setrect0 \
			0, 200 + 100 * num, 150, 260 + 100 * num \
		end

	selectbar(0, "地面");
	selectbar(1, "墙");
	selectbar(2, "检查点");
	selectbar(3, "玩家位置");
	selectbar(4, "箱子");
#undef selectbar

	/************************  编辑界面-右侧  ************************/
	//0
	begin_edit

	plot

	setrect0
		1130, 200, 1280, 260
	end

	//1
	begin_edit

	plot

	setrect0
		1130, 300, 1280, 360
	end

	//2 清除
	begin_edit
		if (!rect.PtInRect(cp))return;
		switch (clicktype) {
		case 1:edit->temp.reset(); break;
		default:break;
		}
	plot
		switch (ptype) {
		case 0:
			prect(pDC, rect, 橙色); 
			ptext(pDC, "清除", rect, 橙色暗); break;
		default:break;
		}
	setrect0
		1130, 400, 1280, 460
	end

	//3 放弃更改
	begin_edit
		if (!rect.PtInRect(cp))return;
		switch (clicktype) {
		case 1: char temp[64]; 
		if (edit->Check(temp, 64)) { game.sence = 1; }
			break;
		default:break;
		}
	plot
		switch (ptype) {
		case 0:
			prect(pDC, rect, 橙色); 
			ptext(pDC, "放弃更改", rect, 橙色暗); break;
		default:break;
		}
	setrect0
		1130, 500, 1280, 560
	end

	//4 应用更改
	begin_edit
		if (!rect.PtInRect(cp))return;
		switch (clicktype) {
		case 1:
		char temp[64];
		if (edit->Check(temp, 64)) { edit->printmap(); game.sence = 1; }
		break;
		default:break;
		}
	plot
		switch (ptype) {
		case 0:
			prect(pDC, rect, 橙色); 
			ptext(pDC, "应用更改", rect, 橙色暗); break;
		default:break;
		}
	setrect0
		1130, 600, 1280, 660
	end

	/************************  编辑界面-下方  ************************/
	//地图检查
	begin_edit
	plot
		switch (ptype) {
		case 0:
			prect(pDC, rect, 橙色);
			char temp[64];
			edit->Check(temp, 64);
			ptext(pDC, temp, rect, 橙色暗); break;
		default:break;
		}
	setrect0
		600, 800, 680, 860
	end


	/************************  游戏界面-中心  ************************/
	//背景
	begin_play
		if (!rect.PtInRect(cp))return;
		switch (clicktype) {
		case 2:play->Undo(); play->lastflag = 0; break;
		default:break;
		}
	plot
		switch (ptype) {
		case 0:
			prect(pDC, rect, RGB(0, 120, 215)); break;
		default:break;
		}
	setrect0
		0, 0, 1280, 960
	end

	//中央棋盘
	begin_play
		if (!rect.PtInRect(cp))return;
		P p = pchess_calij(cp, rect, play->map);
		switch (clicktype) {
		case 0:play->Set(p); break;
		case 1:play->Do(p); break;
		case 2:play->Undo(); play->lastflag = 0; break;
		default:break;
		}
	plot
		switch (ptype) {
		case 0:DATA cur; if (play->lastflag)cur = play->lastresult; else cur = play->getcur();
			pchess(pDC, play->map, cur, rect); break;
		default:break;
		}
	setrect0
		200, 200, 1080, 760
	end

	/************************  游戏界面-右侧  ************************/
	//1 自动解算标签
	begin_play
		if (!rect.PtInRect(cp))return;
		switch (clicktype) {
		case 1:play->solvemain(); break;
		default:break;
		}
	plot
		switch (ptype) {
		case 0:
			prect(pDC, rect, 橙色); 
			switch (play->bfslastflag) {
			case 2:ptext(pDC, "自动求解", rect, 橙色暗); break;
			case 1:ptext(pDC, "已解出", rect, 橙色暗); break;
			case 0:ptext(pDC, "此地图无解", rect, 橙色暗); break;
			default:break;
			}
			break;
		default:break;
		}
	setrect0
		1130, 300, 1280, 360
	end

	//4 回到编辑界面
	begin_play
		if (!rect.PtInRect(cp))return;
		switch (clicktype) {
		case 1:
			play->leave();
			game.sence = 0;
		break;
		default:break;
		}
	plot
		switch (ptype) {
		case 0:
			prect(pDC, rect, 橙色);
			ptext(pDC, "返回编辑", rect, 橙色暗); break;
		default:break;
		}
	setrect0
		1130, 600, 1280, 660
	end

	/************************  游戏界面-下方  ************************/
	//解出
	begin_play
	plot
		switch (ptype) {
		case 0:if (!play->map.isend(play->getcur()))break;
			prect(pDC, rect, 橙色);
			ptext(pDC, "好！ 已经解出地图 按右键可以逐步回退", rect, 橙色暗); break;
		default:break;
		}
	setrect0
		600, 800, 680, 860
	end

	//自动解算 进度条
	begin_play
	plot
		switch (ptype) {
		case 0:if (play->sence == 0)break;
			pcalbar(pDC, play->bfsinfo, rect);
			break;
		default:break;
		}
	setrect0
		340, 800, 940, 860
	end

#undef 橙色暗
#undef 橙色
#undef begin_edit
#undef begin_play
#undef plot
#undef setrect0
#undef end
}

#undef args_core
#undef args_plot
#undef N

/*****************************************************************        MFC消息反射        *****************************************************************/
//消息反射函数
int CMyFrameWnd::OnCreate(LPCREATESTRUCT p) {
	//TODO: 设置定时器
	SetTimer(1, 10, NULL);
	time_t timems = clock();
	timemsold = timems;
	return CFrameWnd::OnCreate(p);
}
void CMyFrameWnd::OnMouseMove(UINT nKey, CPoint pt) {
	this->pt = pt;
	return CFrameWnd::OnMouseMove(nKey, pt);
}
void CMyFrameWnd::OnLButtonUp(UINT nKey, CPoint pt) {
	this->nKey = nKey;
	return CFrameWnd::OnLButtonUp(nKey, pt);
}
void CMyFrameWnd::OnLButtonDown(UINT nKey, CPoint pt) {
	this->nKey = nKey;
	return CFrameWnd::OnLButtonDown(nKey, pt);
}
void CMyFrameWnd::OnRButtonUp(UINT nKey, CPoint pt) {
	this->nKey = nKey;
	game.inputmain(pt, nKey);
	return CFrameWnd::OnRButtonUp(nKey, pt);
}
void CMyFrameWnd::OnRButtonDown(UINT nKey, CPoint pt) {
	this->nKey = nKey;
	game.inputmain(pt, nKey);
	return CFrameWnd::OnRButtonDown(nKey, pt);
}
void CMyFrameWnd::OnTimer(UINT_PTR ptr) {
	//TODO: 在其他需要重绘的反射中添加重绘
	game.inputmain(pt, nKey);
	CRect temp; temp.SetRect(0, 0, screensize.cx, screensize.cy);
	::InvalidateRect(this->m_hWnd, &temp, FALSE);
	return CFrameWnd::OnTimer(ptr);
}
void CMyFrameWnd::OnSize(UINT uint, int x, int y) {
	screensize.cx = x; screensize.cy = y;
	return CFrameWnd::OnSize(uint, x, y);
}

//双缓冲刷新
void CMyFrameWnd::doublebuffer(CDC *pDC) {
	//定义对象
	CDC MemDC; CBitmap MemBitmap;
	//使用函数初始化1：建立与屏幕显示兼容的内存显示设备
	MemDC.CreateCompatibleDC(NULL);
	//使用函数初始化2：建立与屏幕显示兼容的位图
	MemBitmap.CreateCompatibleBitmap(pDC, screensize.cx, screensize.cy);
	//将位图与内存显示设备进行链接
	CBitmap *pOldBit = MemDC.SelectObject(&MemBitmap);
	//调用功能：用背景色将位图清除干净 TODO: 设置清屏颜色
	MemDC.FillSolidRect(0, 0, screensize.cx, screensize.cy, RGB(196, 196, 240));

	//绘图
	timems = clock();

	game.scalemain(screensize);
	game.plotmain(&MemDC);
	//TODO: 绘图部分
	timemsold = timems;

	//位图复制：从显示设备复制到主显示设备PDC 此时刷新画面
	pDC->BitBlt(0, 0, screensize.cx, screensize.cy, &MemDC, 0, 0, SRCCOPY);
	//使用函数清理：清理两个对象
	MemBitmap.DeleteObject();
	MemDC.DeleteDC();
	return;
}
//绘图消息反射
void CMyFrameWnd::OnPaint() {
	CDC *pDC = GetDC();
	doublebuffer(pDC);
	this->ReleaseDC(pDC);
	return CFrameWnd::OnPaint();
}
