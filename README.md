theme: jekyll-theme-minimal
title: Octocat's homepage
description: Bookmark this to keep an eye on my project updates!
# xavier-chien.github.io
測試用
<!DOCTYPE html>
<html lang="zh-Hant">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>待辦清單 (Todo List) - Local Storage 版</title>
  
  <!-- 引入 Tailwind CSS 確保響應式設計和美觀 -->
  <script src="https://cdn.tailwindcss.com"></script>
  <!-- 引入 React, ReactDOM, 和 Babel 進行前端組件化 -->
  <script src="https://unpkg.com/react@18/umd/react.development.js"></script>
  <script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
  
  <!-- 配置 Tailwind 的字體和樣式 -->
  <style>
    /* 確保所有元素使用 Inter 字體 */
    @import url('https://fonts.googleapis.com/css2?family=Inter:wght@100..900&display=swap');
    body {
      font-family: 'Inter', sans-serif;
    }
  </style>
</head>
<body class="bg-gray-100 min-h-screen">

<div id="root"></div>

<script type="text/babel">
  // 引入 React 核心功能
  const { useState, useEffect, useCallback } = React;
  // 引入 Lucide icons (圖標)
  const { ListChecks, Plus, Trash2, CheckCircle, Circle } = window['lucide']; 

  // Local Storage Key，用於儲存待辦事項列表
  const LOCAL_STORAGE_KEY = 'todoListAppTasks';

  // --- 輔助函數：處理 Local Storage 讀寫 ---
  
  // 1. 讀取待辦事項
  const getTodosFromStorage = () => {
    try {
      const savedTasks = localStorage.getItem(LOCAL_STORAGE_KEY);
      // 讀取並解析 JSON 數據，如果無數據則返回空陣列
      return savedTasks ? JSON.parse(savedTasks) : [];
    } catch (error) {
      console.error("讀取 Local Storage 失敗:", error);
      return [];
    }
  };

  // 2. 儲存待辦事項
  const saveTodosToStorage = (todos) => {
    try {
      localStorage.setItem(LOCAL_STORAGE_KEY, JSON.stringify(todos));
      console.log(`待辦清單已儲存。總數: ${todos.length}`);
    } catch (error) {
      console.error("寫入 Local Storage 失敗:", error);
    }
  };

  // --- Todo 列表項目組件 ---
  const TodoItem = React.memo(({ todo, onToggle, onDelete }) => (
    <div className={`flex items-center p-3 sm:p-4 rounded-lg shadow-sm mb-3 transition duration-200 ease-in-out border 
                    ${todo.completed ? 'bg-green-50 border-green-200' : 'bg-white border-gray-200 hover:shadow-md'}`}>
      
      {/* 標記完成按鈕 */}
      <button 
        onClick={() => onToggle(todo.id)}
        className="w-8 h-8 flex items-center justify-center text-gray-400 hover:text-indigo-600 transition duration-150 mr-3 flex-shrink-0"
        aria-label={todo.completed ? '標記為未完成' : '標記為完成'}
      >
        {todo.completed 
          ? <CheckCircle className="w-6 h-6 text-green-500 fill-green-500" /> 
          : <Circle className="w-6 h-6" />
        }
      </button>

      {/* 任務文字 */}
      <span className={`flex-grow text-lg sm:text-xl font-medium break-words
                       ${todo.completed ? 'line-through text-gray-400' : 'text-gray-700'}`}>
        {todo.text}
      </span>

      {/* 刪除按鈕 */}
      <button 
        onClick={() => onDelete(todo.id)}
        className="ml-4 p-2 rounded-full text-red-400 hover:bg-red-100 hover:text-red-600 transition duration-150 flex-shrink-0"
        aria-label="刪除任務"
      >
        <Trash2 className="w-5 h-5" />
      </button>
    </div>
  ));

  // --- 應用程式主組件 ---
  const App = () => {
    const [todos, setTodos] = useState(getTodosFromStorage);
    const [newTodoText, setNewTodoText] = useState('');

    // 數據同步：當 todos 陣列改變時，自動將新值寫入 Local Storage
    useEffect(() => {
      saveTodosToStorage(todos);
    }, [todos]); 

    // 處理新增待辦事項
    const handleAddTodo = useCallback((e) => {
      e.preventDefault();
      const trimmedText = newTodoText.trim();
      if (!trimmedText) return;

      const newTodo = {
        // 為了確保 ID 唯一性，使用時間戳或更好的 UUID
        id: Date.now(), 
        text: trimmedText,
        completed: false,
      };

      setTodos(prevTodos => [newTodo, ...prevTodos]); // 新的放前面
      setNewTodoText(''); // 清空輸入欄
    }, [newTodoText]);

    // 處理切換完成狀態
    const handleToggleTodo = useCallback((id) => {
      setTodos(prevTodos => 
        prevTodos.map(todo => 
          todo.id === id ? { ...todo, completed: !todo.completed } : todo
        )
      );
    }, []);

    // 處理刪除待辦事項
    const handleDeleteTodo = useCallback((id) => {
      setTodos(prevTodos => prevTodos.filter(todo => todo.id !== id));
    }, []);

    // 計算任務統計
    const totalTasks = todos.length;
    const completedTasks = todos.filter(t => t.completed).length;

    return (
      <div className="min-h-screen flex flex-col items-center justify-start py-8 px-4">
        <div className="w-full max-w-2xl bg-white shadow-2xl rounded-2xl p-6 md:p-10 border-t-8 border-indigo-600">
          
          {/* 標題與統計區 */}
          <h1 className="text-3xl sm:text-4xl font-extrabold text-center text-indigo-700 mb-2">
            <ListChecks className="inline w-8 h-8 mr-2 text-indigo-500" />
            個人待辦清單
          </h1>
          <p className="text-center text-gray-500 mb-6">
            您專屬的純前端任務管理器 (Local Storage 持久化)
          </p>
          
          {/* 統計面板 */}
          <div className="bg-indigo-50 p-4 rounded-lg mb-8 shadow-inner flex justify-around text-center">
            <div>
              <p className="text-2xl font-bold text-indigo-800">{totalTasks}</p>
              <p className="text-sm text-gray-600">總任務數</p>
            </div>
            <div>
              <p className="text-2xl font-bold text-green-600">{completedTasks}</p>
              <p className="text-sm text-gray-600">已完成</p>
            </div>
            <div>
              <p className="text-2xl font-bold text-red-600">{totalTasks - completedTasks}</p>
              <p className="text-sm text-gray-600">待辦中</p>
            </div>
          </div>

          {/* 新增任務表單 */}
          <form onSubmit={handleAddTodo} className="flex space-x-2 mb-8">
            <input
              type="text"
              value={newTodoText}
              onChange={(e) => setNewTodoText(e.target.value)}
              placeholder="輸入新的待辦事項..."
              className="flex-grow p-3 border-2 border-gray-300 rounded-lg focus:outline-none focus:border-indigo-500 text-gray-700 transition duration-150"
              aria-label="新的待辦事項輸入"
            />
            <button
              type="submit"
              className="p-3 bg-indigo-600 text-white rounded-lg shadow-md hover:bg-indigo-700 transition duration-200 transform hover:scale-105 flex items-center justify-center"
              aria-label="新增任務"
            >
              <Plus className="w-6 h-6" />
            </button>
          </form>

          {/* 待辦事項列表 */}
          <div className="space-y-3">
            {todos.length > 0 ? (
              todos.map(todo => (
                <TodoItem 
                  key={todo.id} 
                  todo={todo} 
                  onToggle={handleToggleTodo} 
                  onDelete={handleDeleteTodo} 
                />
              ))
            ) : (
              <div className="text-center p-10 bg-gray-50 rounded-lg border border-dashed border-gray-300">
                <p className="text-lg text-gray-500 font-medium">🎉 恭喜您，目前沒有任何待辦事項！</p>
                <p className="text-sm text-gray-400 mt-2">在上方輸入框中新增您的第一個任務吧。</p>
              </div>
            )}
          </div>
          
        </div>
      </div>
    );
  };

  // 渲染主 APP
  window.onload = () => {
    // 確保 Lucide 圖標庫已載入
    if (window['lucide']) {
        ReactDOM.createRoot(document.getElementById('root')).render(<App />);
    } else {
        console.error("Lucide icons 庫未能載入。");
    }
  };
</script>

<!-- Lucide Icons 載入 (用於 React 組件) -->
<script src="https://unpkg.com/lucide@latest"></script>

</body>
</html>
