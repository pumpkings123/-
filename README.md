# -import React, { useState, useEffect, useMemo } from 'react';
import { 
  CheckCircle, 
  Circle, 
  Droplet, 
  Plus, 
  Trash2, 
  BarChart2, 
  Calendar, 
  Settings, 
  Clock, 
  Sun, 
  Moon, 
  Coffee, 
  Utensils, 
  Briefcase, 
  BookOpen, 
  Activity,
  Bell,
  X
} from 'lucide-react';
import { 
  BarChart, 
  Bar, 
  XAxis, 
  YAxis, 
  Tooltip, 
  ResponsiveContainer, 
  CartesianGrid 
} from 'recharts';

// --- 配置与常量 ---

const THEMES = {
  ocean: {
    name: '深海蓝',
    bg: 'bg-slate-50',
    primary: 'bg-blue-600',
    primaryLight: 'bg-blue-100',
    text: 'text-slate-800',
    accent: 'text-blue-600',
    card: 'bg-white'
  },
  nature: {
    name: '森林绿',
    bg: 'bg-green-50',
    primary: 'bg-emerald-600',
    primaryLight: 'bg-emerald-100',
    text: 'text-stone-800',
    accent: 'text-emerald-600',
    card: 'bg-white'
  },
  sunset: {
    name: '夕阳红',
    bg: 'bg-orange-50',
    primary: 'bg-orange-500',
    primaryLight: 'bg-orange-100',
    text: 'text-gray-800',
    accent: 'text-orange-500',
    card: 'bg-white'
  },
  dark: {
    name: '暗夜黑',
    bg: 'bg-gray-900',
    primary: 'bg-indigo-500',
    primaryLight: 'bg-gray-800',
    text: 'text-gray-100',
    accent: 'text-indigo-400',
    card: 'bg-gray-800'
  }
};

const CATEGORIES = [
  { id: 'meal', label: '饮食', icon: <Utensils size={16} />, color: 'text-orange-500' },
  { id: 'work', label: '工作', icon: <Briefcase size={16} />, color: 'text-blue-500' },
  { id: 'study', label: '学习', icon: <BookOpen size={16} />, color: 'text-purple-500' },
  { id: 'health', label: '健康', icon: <Activity size={16} />, color: 'text-green-500' },
  { id: 'other', label: '其他', icon: <Circle size={16} />, color: 'text-gray-500' },
];

// --- 模拟数据 ---
const INITIAL_TASKS = [
  { id: 1, title: '晨间瑜伽', time: '07:00', category: 'health', completed: true },
  { id: 2, title: '营养早餐', time: '07:30', category: 'meal', completed: true },
  { id: 3, title: '团队早会', time: '09:00', category: 'work', completed: false },
  { id: 4, title: '深度工作段', time: '10:00', category: 'work', completed: false },
  { id: 5, title: '午餐休息', time: '12:00', category: 'meal', completed: false },
];

const INITIAL_HABITS = [
  { id: 1, title: '阅读30分钟', streak: 12, completedToday: false },
  { id: 2, title: '不吃宵夜', streak: 5, completedToday: true },
  { id: 3, title: '冥想', streak: 20, completedToday: false },
];

// --- 组件 ---

export default function DailyLifePlanner() {
  const [activeTab, setActiveTab] = useState('dashboard');
  const [currentTheme, setCurrentTheme] = useState('ocean');
  const [tasks, setTasks] = useState(INITIAL_TASKS);
  const [waterCount, setWaterCount] = useState(3);
  const [habits, setHabits] = useState(INITIAL_HABITS);
  const [notification, setNotification] = useState(null);
  
  // 新建任务表单状态
  const [showAddModal, setShowAddModal] = useState(false);
  const [newTask, setNewTask] = useState({ title: '', time: '', category: 'work' });

  const theme = THEMES[currentTheme];

  // 模拟通知检查 (每分钟检查一次)
  useEffect(() => {
    const checkReminders = () => {
      const now = new Date();
      const currentTime = `${String(now.getHours()).padStart(2, '0')}:${String(now.getMinutes()).padStart(2, '0')}`;
      
      const taskDue = tasks.find(t => t.time === currentTime && !t.completed);
      if (taskDue) {
        showNotification(`提醒: 该开始 "${taskDue.title}" 了！`);
      }
      
      // 喝水提醒 (简单的每2小时逻辑模拟)
      if (['10:00', '14:00', '16:00'].includes(currentTime)) {
        showNotification("💧 该喝水了！保持水分充足。");
      }
    };

    const interval = setInterval(checkReminders, 60000); // 每一分钟
    return () => clearInterval(interval);
  }, [tasks]);

  const showNotification = (msg) => {
    setNotification(msg);
    setTimeout(() => setNotification(null), 4000);
  };

  // --- 逻辑处理函数 ---

  const handleAddTask = () => {
    if (!newTask.title || !newTask.time) return;
    const task = {
      id: Date.now(),
      ...newTask,
      completed: false
    };
    setTasks([...tasks, task].sort((a, b) => a.time.localeCompare(b.time)));
    setShowAddModal(false);
    setNewTask({ title: '', time: '', category: 'work' });
    showNotification("任务已添加！");
  };

  const toggleTask = (id) => {
    setTasks(tasks.map(t => t.id === id ? { ...t, completed: !t.completed } : t));
  };

  const deleteTask = (id) => {
    setTasks(tasks.filter(t => t.id !== id));
  };

  const addWater = () => {
    setWaterCount(prev => Math.min(prev + 1, 10));
    if(waterCount + 1 === 8) showNotification("太棒了！今日饮水目标达成 🎉");
  };

  const toggleHabit = (id) => {
    setHabits(habits.map(h => {
      if (h.id === id) {
        const isCompleted = !h.completedToday;
        return {
          ...h,
          completedToday: isCompleted,
          streak: isCompleted ? h.streak + 1 : h.streak - 1
        };
      }
      return h;
    }));
  };

  // --- 统计数据生成 ---
  const statsData = useMemo(() => {
    const completedTasks = tasks.filter(t => t.completed).length;
    const totalTasks = tasks.length;
    const completionRate = totalTasks ? Math.round((completedTasks / totalTasks) * 100) : 0;
    
    return [
      { name: '任务完成', value: completionRate },
      { name: '水分摄入', value: (waterCount / 8) * 100 },
      { name: '习惯打卡', value: (habits.filter(h => h.completedToday).length / habits.length) * 100 },
    ];
  }, [tasks, waterCount, habits]);

  // --- 子组件渲染 ---

  const renderDashboard = () => (
    <div className="space-y-6 animate-fade-in">
      {/* 顶部欢迎卡片 */}
      <div className={`${theme.primary} rounded-2xl p-6 text-white shadow-lg`}>
        <h2 className="text-2xl font-bold mb-1">早安，规划师！</h2>
        <p className="opacity-90 text-sm mb-4">今天也要元气满满哦。</p>
        <div className="flex justify-between items-end">
          <div>
            <div className="text-3xl font-bold">{tasks.filter(t => t.completed).length}/{tasks.length}</div>
            <div className="text-xs opacity-80">今日任务</div>
          </div>
          <div className="bg-white/20 p-2 rounded-lg backdrop-blur-sm">
             <div className="text-xl font-bold flex items-center gap-1">
               {waterCount}<span className="text-xs font-normal">/8杯</span>
             </div>
             <div className="text-xs opacity-80">饮水进度</div>
          </div>
        </div>
      </div>

      {/* 快速操作区 */}
      <div className="grid grid-cols-2 gap-4">
        <div 
          onClick={() => setActiveTab('water')}
          className={`${theme.card} p-4 rounded-xl shadow-sm border border-gray-100 flex items-center justify-between cursor-pointer hover:shadow-md transition-all`}>
          <div className="flex items-center gap-3">
            <div className="bg-blue-100 p-2 rounded-full text-blue-500"><Droplet size={20}/></div>
            <div>
              <div className={`font-semibold ${theme.text}`}>补水</div>
              <div className="text-xs text-gray-400">点击记录</div>
            </div>
          </div>
          <Plus size={16} className="text-gray-400"/>
        </div>
        <div 
           onClick={() => setShowAddModal(true)}
           className={`${theme.card} p-4 rounded-xl shadow-sm border border-gray-100 flex items-center justify-between cursor-pointer hover:shadow-md transition-all`}>
          <div className="flex items-center gap-3">
            <div className="bg-purple-100 p-2 rounded-full text-purple-500"><Calendar size={20}/></div>
            <div>
              <div className={`font-semibold ${theme.text}`}>加任务</div>
              <div className="text-xs text-gray-400">安排日程</div>
            </div>
          </div>
          <Plus size={16} className="text-gray-400"/>
        </div>
      </div>

      {/* 近期任务 */}
      <div>
        <div className="flex justify-between items-center mb-3">
          <h3 className={`font-bold text-lg ${theme.text}`}>近期日程</h3>
          <button onClick={() => setActiveTab('planner')} className={`text-xs ${theme.accent} font-medium`}>查看全部</button>
        </div>
        <div className="space-y-3">
          {tasks.slice(0, 3).map(task => (
             <TaskItem key={task.id} task={task} toggle={toggleTask} theme={theme} />
          ))}
        </div>
      </div>
    </div>
  );

  const renderPlanner = () => (
    <div className="space-y-4 pb-20">
      <div className="flex justify-between items-center mb-2">
        <h2 className={`text-2xl font-bold ${theme.text}`}>今日日程</h2>
        <button 
          onClick={() => setShowAddModal(true)}
          className={`${theme.primary} text-white px-4 py-2 rounded-lg text-sm font-medium flex items-center gap-2 shadow-md active:scale-95 transition-transform`}>
          <Plus size={16} /> 新增
        </button>
      </div>

      <div className="space-y-3">
        {tasks.map((task) => (
          <div key={task.id} className="group relative">
             <TaskItem task={task} toggle={toggleTask} theme={theme} fullDetails />
             <button 
                onClick={() => deleteTask(task.id)}
                className="absolute right-4 top-1/2 -translate-y-1/2 text-red-400 opacity-0 group-hover:opacity-100 transition-opacity p-2 bg-red-50 rounded-full">
                <Trash2 size={16} />
             </button>
          </div>
        ))}
        {tasks.length === 0 && (
          <div className="text-center py-10 text-gray-400">
            暂无安排，去添加一个吧！
          </div>
        )}
      </div>
    </div>
  );

  const renderWaterAndHabits = () => (
    <div className="space-y-6">
      {/* 水分追踪 */}
      <div className={`${theme.card} p-6 rounded-2xl shadow-sm border border-gray-100 text-center`}>
        <h3 className={`text-xl font-bold mb-6 ${theme.text}`}>水分追踪</h3>
        <div className="relative w-40 h-40 mx-auto mb-6">
           {/* 圆形进度模拟 */}
           <svg className="w-full h-full transform -rotate-90">
             <circle cx="80" cy="80" r="70" stroke="#eee" strokeWidth="12" fill="transparent" />
             <circle 
                cx="80" cy="80" r="70" 
                stroke="#3b82f6" strokeWidth="12" 
                fill="transparent"
                strokeDasharray={440}
                strokeDashoffset={440 - (440 * waterCount) / 10}
                className="transition-all duration-1000 ease-out"
             />
           </svg>
           <div className="absolute inset-0 flex flex-col items-center justify-center">
             <span className="text-4xl font-bold text-blue-500">{waterCount}</span>
             <span className="text-xs text-gray-400">/ 8 杯</span>
           </div>
        </div>
        <div className="flex justify-center gap-4">
          <button 
            onClick={() => setWaterCount(Math.max(0, waterCount - 1))}
            className="w-10 h-10 rounded-full bg-gray-100 flex items-center justify-center text-gray-500 hover:bg-gray-200">
            -
          </button>
          <button 
            onClick={addWater}
            className="w-10 h-10 rounded-full bg-blue-500 text-white flex items-center justify-center shadow-lg hover:bg-blue-600 active:scale-95 transition-transform">
            <Plus size={24} />
          </button>
        </div>
      </div>

      {/* 习惯追踪 */}
      <div>
        <h3 className={`text-xl font-bold mb-4 ${theme.text}`}>习惯打卡</h3>
        <div className="grid gap-3">
          {habits.map(habit => (
            <div key={habit.id} className={`${theme.card} p-4 rounded-xl shadow-sm flex justify-between items-center border border-gray-100`}>
              <div className="flex items-center gap-3">
                <div onClick={() => toggleHabit(habit.id)} className="cursor-pointer">
                  {habit.completedToday 
                    ? <CheckCircle className="text-green-500" size={24} />
                    : <Circle className="text-gray-300" size={24} />
                  }
                </div>
                <div>
                  <div className={`font-medium ${habit.completedToday ? 'line-through text-gray-400' : theme.text}`}>{habit.title}</div>
                  <div className="text-xs text-orange-400 flex items-center gap-1">
                     🔥 连续坚持 {habit.streak} 天
                  </div>
                </div>
              </div>
            </div>
          ))}
        </div>
      </div>
    </div>
  );

  const renderStats = () => (
    <div className="space-y-6">
      <h2 className={`text-2xl font-bold ${theme.text}`}>进度统计</h2>
      
      <div className={`${theme.card} p-6 rounded-2xl shadow-sm border border-gray-100 h-64`}>
        <h3 className="text-sm font-bold text-gray-400 mb-4 uppercase tracking-wider">今日完成率</h3>
        <ResponsiveContainer width="100%" height="100%">
          <BarChart data={statsData}>
            <CartesianGrid strokeDasharray="3 3" vertical={false} />
            <XAxis dataKey="name" tick={{fontSize: 12}} />
            <YAxis hide />
            <Tooltip 
              cursor={{fill: 'transparent'}}
              contentStyle={{ borderRadius: '8px', border: 'none', boxShadow: '0 4px 12px rgba(0,0,0,0.1)' }}
            />
            <Bar dataKey="value" fill={currentTheme === 'dark' ? '#6366f1' : '#3b82f6'} radius={[6, 6, 0, 0]} barSize={40} />
          </BarChart>
        </ResponsiveContainer>
      </div>

      <div className={`${theme.card} p-6 rounded-2xl shadow-sm border border-gray-100`}>
        <h3 className="text-lg font-bold mb-4">个性化设置</h3>
        <div className="grid grid-cols-2 gap-3">
          {Object.entries(THEMES).map(([key, t]) => (
            <button
              key={key}
              onClick={() => setCurrentTheme(key)}
              className={`p-3 rounded-lg border-2 text-left transition-all ${currentTheme === key ? 'border-blue-500 bg-blue-50' : 'border-transparent bg-gray-100'}`}
            >
              <div className={`w-full h-8 rounded mb-2 ${t.primary}`}></div>
              <div className="text-sm font-medium text-gray-700">{t.name}</div>
            </button>
          ))}
        </div>
      </div>
    </div>
  );

  return (
    <div className={`min-h-screen ${theme.bg} transition-colors duration-300 font-sans pb-24 md:pb-0`}>
      {/* 桌面端布局容器 */}
      <div className="max-w-md mx-auto md:max-w-4xl md:grid md:grid-cols-[240px_1fr] min-h-screen shadow-2xl overflow-hidden bg-white/50 backdrop-blur-sm">
        
        {/* 侧边栏 (桌面) */}
        <div className="hidden md:flex flex-col border-r bg-white p-6 justify-between">
           <div>
             <h1 className="text-2xl font-extrabold text-transparent bg-clip-text bg-gradient-to-r from-blue-500 to-teal-400 mb-10">
               LifePlan.
             </h1>
             <nav className="space-y-2">
               <NavButton icon={<Activity />} label="概览" active={activeTab === 'dashboard'} onClick={() => setActiveTab('dashboard')} theme={theme} />
               <NavButton icon={<Calendar />} label="日程" active={activeTab === 'planner'} onClick={() => setActiveTab('planner')} theme={theme} />
               <NavButton icon={<Droplet />} label="健康" active={activeTab === 'water'} onClick={() => setActiveTab('water')} theme={theme} />
               <NavButton icon={<BarChart2 />} label="统计" active={activeTab === 'stats'} onClick={() => setActiveTab('stats')} theme={theme} />
             </nav>
           </div>
           <div className="text-xs text-gray-400">© 2023 Daily Planner</div>
        </div>

        {/* 主要内容区域 */}
        <main className="relative h-screen overflow-y-auto no-scrollbar">
           {/* 移动端顶部 Header */}
           <div className="md:hidden sticky top-0 z-10 bg-white/80 backdrop-blur-md px-6 py-4 border-b flex justify-between items-center">
             <span className="font-bold text-lg text-slate-800">LifePlan.</span>
             <div className="flex gap-2">
                <button onClick={() => setCurrentTheme(currentTheme === 'dark' ? 'ocean' : 'dark')} className="p-2 rounded-full bg-gray-100">
                  {currentTheme === 'dark' ? <Sun size={18}/> : <Moon size={18}/>}
                </button>
             </div>
           </div>

           {/* 动态通知 Toast */}
           {notification && (
             <div className="fixed top-20 left-1/2 -translate-x-1/2 z-50 bg-slate-800 text-white px-4 py-2 rounded-full shadow-lg text-sm flex items-center gap-2 animate-bounce-in">
               <Bell size={14} className="text-yellow-400"/> {notification}
             </div>
           )}

           <div className="p-6">
             {activeTab === 'dashboard' && renderDashboard()}
             {activeTab === 'planner' && renderPlanner()}
             {activeTab === 'water' && renderWaterAndHabits()}
             {activeTab === 'stats' && renderStats()}
           </div>
        </main>
      </div>

      {/* 移动端底部导航栏 */}
      <div className="md:hidden fixed bottom-0 left-0 right-0 bg-white border-t border-gray-100 px-6 py-3 flex justify-between items-center z-20 safe-area-bottom">
         <MobileNavItem icon={<Activity />} active={activeTab === 'dashboard'} onClick={() => setActiveTab('dashboard')} theme={theme} />
         <MobileNavItem icon={<Calendar />} active={activeTab === 'planner'} onClick={() => setActiveTab('planner')} theme={theme} />
         <div className="relative -top-5">
           <button 
             onClick={() => setShowAddModal(true)}
             className={`w-12 h-12 rounded-full ${theme.primary} text-white flex items-center justify-center shadow-lg shadow-blue-500/30 transform active:scale-95 transition-all`}>
             <Plus size={24} />
           </button>
         </div>
         <MobileNavItem icon={<Droplet />} active={activeTab === 'water'} onClick={() => setActiveTab('water')} theme={theme} />
         <MobileNavItem icon={<BarChart2 />} active={activeTab === 'stats'} onClick={() => setActiveTab('stats')} theme={theme} />
      </div>

      {/* 新增任务 Modal */}
      {showAddModal && (
        <div className="fixed inset-0 z-50 flex items-center justify-center bg-black/40 backdrop-blur-sm p-4">
          <div className={`${theme.card} w-full max-w-sm rounded-2xl p-6 shadow-2xl animate-scale-up`}>
            <div className="flex justify-between items-center mb-6">
              <h3 className={`text-xl font-bold ${theme.text}`}>添加新活动</h3>
              <button onClick={() => setShowAddModal(false)} className="text-gray-400 hover:text-gray-600">
                <X size={24} />
              </button>
            </div>
            
            <div className="space-y-4">
              <div>
                <label className="block text-sm font-medium text-gray-500 mb-1">活动内容</label>
                <input 
                  type="text" 
                  value={newTask.title}
                  onChange={(e) => setNewTask({...newTask, title: e.target.value})}
                  placeholder="例如: 晨跑、吃午餐..."
                  className={`w-full p-3 rounded-lg border bg-gray-50 focus:outline-none focus:ring-2 focus:ring-blue-500 ${theme.text}`}
                />
              </div>
              
              <div className="grid grid-cols-2 gap-4">
                <div>
                   <label className="block text-sm font-medium text-gray-500 mb-1">时间</label>
                   <input 
                    type="time" 
                    value={newTask.time}
                    onChange={(e) => setNewTask({...newTask, time: e.target.value})}
                    className="w-full p-3 rounded-lg border bg-gray-50 focus:outline-none focus:ring-2 focus:ring-blue-500"
                  />
                </div>
                <div>
                   <label className="block text-sm font-medium text-gray-500 mb-1">分类</label>
                   <select 
                     value={newTask.category}
                     onChange={(e) => setNewTask({...newTask, category: e.target.value})}
                     className="w-full p-3 rounded-lg border bg-gray-50 focus:outline-none focus:ring-2 focus:ring-blue-500">
                     {CATEGORIES.map(c => <option key={c.id} value={c.id}>{c.label}</option>)}
                   </select>
                </div>
              </div>

              <div className="pt-4">
                <button 
                  onClick={handleAddTask}
                  className={`w-full py-3 rounded-xl font-bold text-white shadow-lg ${theme.primary} active:opacity-90 transition-opacity`}>
                  确认添加
                </button>
              </div>
            </div>
          </div>
        </div>
      )}
    </div>
  );
}

// --- 辅助小组件 ---

const TaskItem = ({ task, toggle, theme, fullDetails = false }) => {
  const category = CATEGORIES.find(c => c.id === task.category) || CATEGORIES[4];
  
  return (
    <div className={`${theme.card} p-4 rounded-xl shadow-sm border-l-4 ${task.completed ? 'border-gray-300 opacity-60' : 'border-blue-500'} flex items-center justify-between transition-all hover:shadow-md bg-white`}>
      <div className="flex items-center gap-3">
        <div onClick={() => toggle(task.id)} className="cursor-pointer">
           {task.completed 
             ? <CheckCircle className="text-gray-400" size={20}/> 
             : <Circle className={theme.accent} size={20}/> 
           }
        </div>
        <div>
          <div className={`font-medium ${task.completed ? 'line-through text-gray-400' : theme.text}`}>
            {task.title}
          </div>
          <div className="flex items-center gap-2 text-xs text-gray-400 mt-1">
             <Clock size={12}/> {task.time}
             {fullDetails && (
               <span className={`flex items-center gap-1 ml-2 ${category.color}`}>
                 {category.icon} {category.label}
               </span>
             )}
          </div>
        </div>
      </div>
    </div>
  );
};

const NavButton = ({ icon, label, active, onClick, theme }) => (
  <button 
    onClick={onClick}
    className={`w-full flex items-center gap-3 px-4 py-3 rounded-xl transition-colors ${active ? `${theme.primaryLight} ${theme.accent} font-bold` : 'text-gray-500 hover:bg-gray-100'}`}>
    {icon}
    <span>{label}</span>
  </button>
);

const MobileNavItem = ({ icon, active, onClick, theme }) => (
  <button 
    onClick={onClick}
    className={`p-2 rounded-xl transition-all ${active ? theme.accent : 'text-gray-400'}`}>
    {React.cloneElement(icon, { size: 24, strokeWidth: active ? 2.5 : 2 })}
  </button>
);

