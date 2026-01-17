import React, { useState, useEffect } from 'react';
import { Copy, Check, RefreshCw, User, MessageSquare, ShieldCheck, Phone, Mail, Lock, LogOut, Trash2, Search, AlertCircle, X } from 'lucide-react';
import { initializeApp } from 'firebase/app';
import { getAuth, signInAnonymously, onAuthStateChanged, signInWithCustomToken } from 'firebase/auth';
import { getFirestore, collection, addDoc, onSnapshot, deleteDoc, doc, query } from 'firebase/firestore';

// ============================================================================
// --- Firebase 設定 ---
// ============================================================================

const isCustomConfig = true; // ★ 已啟用您的自訂設定

const myFirebaseConfig = {
  apiKey: "AIzaSyBZpemWzG-OnxKc-AzDVn20nrPGnler8Ho",
  authDomain: "project-6295196669844651558.firebaseapp.com",
  projectId: "project-6295196669844651558",
  storageBucket: "project-6295196669844651558.firebasestorage.app",
  messagingSenderId: "588319438113",
  appId: "1:588319438113:web:b41c7f43b6752723137a60",
  measurementId: "G-ZRJJ0DY8KF"
};

// ============================================================================
// --- 初始化邏輯 (系統自動判斷) ---
// ============================================================================

let firebaseConfig;
let appId = 'default-app-id';

if (isCustomConfig) {
  firebaseConfig = myFirebaseConfig;
  appId = myFirebaseConfig.projectId;
} else {
  try {
    firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : { apiKey: "demo-mode" };
    appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
  } catch (e) {
    console.error("Firebase config error", e);
  }
}

// 初始化 Firebase
const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getFirestore(app);

export default function App() {
  // --- 狀態管理 ---
  const [user, setUser] = useState(null);
  const [view, setView] = useState('user'); // 'user', 'login', 'admin'
  
  // 用戶端狀態
  const [step, setStep] = useState(1);
  const [formData, setFormData] = useState({
    app: '',
    username: '',
    phone: '',
    email: ''
  });
  const [verificationCode, setVerificationCode] = useState('');
  const [copied, setCopied] = useState(false);
  const [error, setError] = useState('');
  const [isSubmitting, setIsSubmitting] = useState(false);

  // 管理員狀態
  const [adminPassword, setAdminPassword] = useState('');
  const [verifications, setVerifications] = useState([]);
  const [searchTerm, setSearchTerm] = useState('');
  
  // 刪除確認視窗狀態
  const [deleteId, setDeleteId] = useState(null);
  const [isDeleting, setIsDeleting] = useState(false);
  const [deleteError, setDeleteError] = useState('');

  // --- Firebase Auth ---
  useEffect(() => {
    const initAuth = async () => {
      if (isCustomConfig) {
        await signInAnonymously(auth);
      } else {
        if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
          await signInWithCustomToken(auth, __initial_auth_token);
        } else {
          await signInAnonymously(auth);
        }
      }
    };
    initAuth().catch(console.error);
    
    const unsubscribe = onAuthStateChanged(auth, setUser);
    return () => unsubscribe();
  }, []);

  // --- 管理員：即時監聽資料 ---
  useEffect(() => {
    if (!user || view !== 'admin') return;

    let collectionRef;
    if (isCustomConfig) {
      collectionRef = collection(db, 'verifications');
    } else {
      collectionRef = collection(db, 'artifacts', appId, 'public', 'data', 'verifications');
    }

    const q = query(collectionRef);
    
    const unsubscribe = onSnapshot(q, (snapshot) => {
      const data = snapshot.docs.map(doc => ({
        id: doc.id,
        ...doc.data()
      }));
      data.sort((a, b) => b.timestamp - a.timestamp);
      setVerifications(data);
    }, (err) => {
      console.error("讀取失敗:", err);
      // 若讀取失敗，顯示紅色錯誤條在最上方
      setDeleteError('讀取失敗：權限不足。請檢查 Firestore Rules。');
    });

    return () => unsubscribe();
  }, [user, view]);

  // --- 應用程式定義 ---
  const apps = [
    { 
      id: 'messenger', 
      name: 'Messenger', 
      color: 'bg-blue-500', 
      textColor: 'text-blue-500',
      fields: ['username'],
      usernameLabel: '暱稱',
      icon: <svg viewBox="0 0 24 24" fill="currentColor" className="w-5 h-5"><path d="M12 2C6.48 2 2 6.03 2 11C2 13.66 3.22 16.03 5.16 17.65C5.35 17.81 5.44 18.06 5.4 18.3L5.16 19.66C5.07 20.18 5.62 20.58 6.09 20.33L8.19 19.23C8.39 19.12 8.62 19.1 8.84 19.16C9.85 19.42 10.91 19.56 12 19.56C17.52 19.56 22 15.53 22 10.56C22 5.59 17.52 2 12 2ZM13.81 14.16L11.45 11.66L6.87 14.16C6.63 14.29 6.36 14.02 6.5 13.78L8.85 9.53L11.21 12.03L15.8 9.53C16.04 9.4 16.31 9.67 16.17 9.91L13.81 14.16Z" /></svg>
    },
    { 
      id: 'line', 
      name: 'LINE', 
      color: 'bg-green-500', 
      textColor: 'text-green-500',
      fields: ['username', 'phone'],
      usernameLabel: '暱稱',
      icon: <svg viewBox="0 0 24 24" fill="currentColor" className="w-5 h-5"><path d="M19.36 10.58C19.36 5.79 14.86 2 9.77 2C4.69 2 0.53 5.61 0.53 10.58C0.53 14.73 3.66 18.23 7.84 19.03C8.16 19.09 8.37 19.28 8.31 19.64L7.96 21.68C7.91 22.03 8.35 22.38 8.76 22.12L13.38 18.96C13.56 18.84 13.75 18.8 13.97 18.8C18.42 18.45 19.36 14.88 19.36 10.58Z" transform="translate(2 1)" /></svg>
    },
    { 
      id: 'discord', 
      name: 'Discord', 
      color: 'bg-indigo-500', 
      textColor: 'text-indigo-500',
      fields: ['username'],
      usernameLabel: '暱稱',
      icon: <svg viewBox="0 0 24 24" fill="currentColor" className="w-5 h-5"><path d="M20.317 4.3698a19.7913 19.7913 0 00-4.8851-1.5152.0741.0741 0 00-.0785.0371c-.211.3753-.4447.8648-.6083 1.2495-1.8447-.2762-3.68-.2762-5.4868 0-.1636-.3933-.4058-.8742-.6177-1.2495a.077.077 0 00-.0785-.037 19.7363 19.7363 0 00-4.8852 1.515.0699.0699 0 00-.0321.0277C.5334 9.0458-.319 13.5799.0992 18.0578a.0824.0824 0 00.0312.0561c2.0528 1.5076 4.0413 2.4228 5.9929 3.0294a.0777.0777 0 00.0842-.0276c.4616-.6304.8731-1.2952 1.226-1.9942a.076.076 0 00-.0416-.1057c-.6528-.2476-1.2743-.5495-1.8722-.8923a.077.077 0 01-.0076-.1277c.1258-.0943.2517-.1923.3718-.2914a.0743.0743 0 01.0776-.0105c3.9278 1.7933 8.18 1.7933 12.0614 0a.0739.0739 0 01.0785.0095c.1202.099.246.1981.3728.2924a.077.077 0 01-.0066.1276 12.2986 12.2986 0 01-1.873.8914.0766.0766 0 00-.0407.1067c.3604.698.7719 1.3628 1.225 1.9932a.076.076 0 00.0842.0286c1.961-.6067 3.9495-1.5219 6.0023-3.0294a.077.077 0 00.0313-.0552c.5004-5.177-.8382-9.6739-3.5485-13.6604a.061.061 0 00-.0312-.0286zM8.02 15.3312c-1.1825 0-2.1569-1.0857-2.1569-2.419 0-1.3332.9555-2.4189 2.157-2.4189 1.2108 0 2.1757 1.0952 2.1568 2.419 0 1.3332-.946 2.419-2.1568 2.419zm7.9748 0c-1.1825 0-2.1569-1.0857-2.1569-2.419 0-1.3332.9554-2.4189 2.1569-2.4189 1.2108 0 2.1757 1.0952 2.1568 2.419 0 1.3332-.946 2.419-2.1568 2.419z"/></svg>
    },
    { 
      id: 'instagram', 
      name: 'Instagram', 
      color: 'bg-pink-600', 
      textColor: 'text-pink-600',
      fields: ['username'],
      usernameLabel: '暱稱',
      icon: <svg viewBox="0 0 24 24" fill="currentColor" className="w-5 h-5"><path d="M12 2.163c3.204 0 3.584.012 4.85.07 3.252.148 4.771 1.691 4.919 4.919.058 1.265.069 1.645.069 4.849 0 3.205-.012 3.584-.069 4.849-.149 3.225-1.664 4.771-4.919 4.919-1.266.058-1.644.07-4.85.07-3.204 0-3.584-.012-4.849-.07-3.26-.149-4.771-1.699-4.919-4.92-.058-1.265-.07-1.644-.07-4.849 0-3.204.013-3.583.07-4.849.149-3.227 1.664-4.771 4.919-4.919 1.266-.057 1.645-.069 4.849-.069zm0-2.163c-3.259 0-3.667.014-4.947.072-4.358.2-6.78 2.618-6.98 6.98-.059 1.281-.073 1.689-.073 4.948 0 3.259.014 3.668.072 4.948.2 4.358 2.618 6.78 6.98 6.98 1.281.058 1.689.072 4.948.072 3.259 0 3.668-.014 4.948-.072 4.354-.2 6.782-2.618 6.979-6.98.059-1.28.073-1.689.073-4.948 0-3.259-.014-3.667-.072-4.947-.196-4.354-2.617-6.78-6.979-6.98-1.281-.059-1.69-.073-4.949-.073zm0 5.838c-3.403 0-6.162 2.759-6.162 6.162s2.759 6.163 6.162 6.163 6.162-2.759 6.162-6.163c0-3.403-2.759-6.162-6.162-6.162zm0 10.162c-2.209 0-4-1.79-4-4 0-2.209 1.791-4 4-4s4 1.791 4 4c0 2.21-1.791 4-4 4zm6.406-11.845c-.796 0-1.441.645-1.441 1.44s.645 1.44 1.441 1.44c.795 0 1.439-.645 1.439-1.44s-.644-1.44-1.439-1.44z"/></svg>
    },
    { 
      id: 'google_chat', 
      name: 'Google Chat', 
      color: 'bg-emerald-600', 
      textColor: 'text-emerald-600',
      fields: ['username', 'email'],
      usernameLabel: '暱稱',
      icon: <svg viewBox="0 0 24 24" fill="currentColor" className="w-5 h-5"><path d="M20 2H4c-1.1 0-2 .9-2 2v18l4-4h14c1.1 0 2-.9 2-2V4c0-1.1-.9-2-2-2z"/><path d="M0 0h24v24H0z" fill="none"/></svg>
    },
  ];

  const selectedAppObj = apps.find(a => a.id === formData.app);

  // --- 邏輯函數 ---
  const generateCode = async () => {
    if (!formData.app) {
      setError('請選擇一個應用程式');
      return;
    }
    if (selectedAppObj?.fields.includes('username') && !formData.username.trim()) {
      setError('請輸入您的暱稱');
      return;
    }
    if (selectedAppObj?.fields.includes('phone') && !formData.phone.trim()) {
      setError('請輸入您的綁定號碼');
      return;
    }
    if (selectedAppObj?.fields.includes('email') && !formData.email.trim()) {
      setError('請輸入您的電子郵件');
      return;
    }
    setError('');
    setIsSubmitting(true);

    const characters = 'ABCDEFGHJKLMNPQRSTUVWXYZ23456789'; 
    const codeLength = Math.floor(Math.random() * 11) + 10;
    let code = '';
    for (let i = 0; i < codeLength; i++) {
      code += characters.charAt(Math.floor(Math.random() * characters.length));
    }
    
    // 儲存到 Firebase
    if (user) {
      try {
        const payload = {
          ...formData,
          code: code,
          timestamp: Date.now(),
          createdAt: new Date().toLocaleString('zh-TW')
        };

        if (isCustomConfig) {
          await addDoc(collection(db, 'verifications'), payload);
        } else {
          await addDoc(collection(db, 'artifacts', appId, 'public', 'data', 'verifications'), payload);
        }

        setVerificationCode(code);
        setStep(2);
      } catch (e) {
        console.error("儲存失敗", e);
        setError('系統連線錯誤，請稍後再試。若您是管理員，請確認 Firestore 權限。');
      }
    } else {
       setError('資料庫尚未連線，請重新整理');
    }
    setIsSubmitting(false);
  };

  const handleCopy = () => {
    let message = `你好，我是 ${formData.username}。`;
    if (formData.phone) message += `\n我的綁定號碼：${formData.phone}`;
    if (formData.email) message += `\n我的電子郵件：${formData.email}`;
    message += `\n這是我的驗證碼：${verificationCode}`;
    
    const textArea = document.createElement("textarea");
    textArea.value = message;
    document.body.appendChild(textArea);
    textArea.select();
    try {
      document.execCommand('copy');
      setCopied(true);
      setTimeout(() => setCopied(false), 2000);
    } catch (err) {
      console.error('Copy failed', err);
    }
    document.body.removeChild(textArea);
  };

  const handleReset = () => {
    setFormData({ app: '', username: '', phone: '', email: '' });
    setVerificationCode('');
    setStep(1);
    setCopied(false);
  };

  const handleLogin = () => {
    if (adminPassword === '20120219') {
      setView('admin');
      setAdminPassword('');
    } else {
      alert('密碼錯誤');
    }
  };

  // --- 新版刪除邏輯：使用自訂 UI ---
  const confirmDelete = (id) => {
    setDeleteId(id);
    setDeleteError('');
  };

  const executeDelete = async () => {
    if (!deleteId) return;
    setIsDeleting(true);
    setDeleteError('');

    try {
      if (isCustomConfig) {
        await deleteDoc(doc(db, 'verifications', deleteId));
      } else {
        await deleteDoc(doc(db, 'artifacts', appId, 'public', 'data', 'verifications', deleteId));
      }
      // 刪除成功，關閉視窗
      setDeleteId(null);
    } catch (e) {
      console.error("刪除失敗", e);
      setDeleteError(e.message);
    } finally {
      setIsDeleting(false);
    }
  };

  // --- Render Views ---

  if (view === 'login') {
    return (
      <div className="min-h-screen bg-gray-100 flex items-center justify-center p-4">
        <div className="bg-white p-8 rounded-2xl shadow-xl w-full max-w-sm">
          <div className="flex justify-center mb-4 text-slate-700">
            <Lock size={40} />
          </div>
          <h2 className="text-xl font-bold text-center mb-6 text-gray-800">管理員登入</h2>
          <input
            type="password"
            value={adminPassword}
            onChange={(e) => setAdminPassword(e.target.value)}
            placeholder="請輸入管理密碼"
            className="w-full p-3 border border-gray-300 rounded-lg mb-4 focus:ring-2 focus:ring-slate-500 outline-none"
            onKeyDown={(e) => e.key === 'Enter' && handleLogin()}
          />
          <button 
            onClick={handleLogin}
            className="w-full bg-slate-800 text-white py-3 rounded-lg font-bold hover:bg-slate-900 transition-colors"
          >
            登入系統
          </button>
          <button 
            onClick={() => setView('user')}
            className="w-full mt-3 text-gray-500 text-sm hover:underline"
          >
            返回用戶驗證頁面
          </button>
        </div>
      </div>
    );
  }

  if (view === 'admin') {
    const filteredVerifications = verifications.filter(v => 
      v.username.toLowerCase().includes(searchTerm.toLowerCase()) || 
      v.code.toLowerCase().includes(searchTerm.toLowerCase())
    );

    return (
      <div className="min-h-screen bg-gray-100 p-4 md:p-8 relative">
        {/* Delete Confirmation Modal */}
        {deleteId && (
          <div className="fixed inset-0 z-50 flex items-center justify-center bg-black/50 backdrop-blur-sm p-4">
            <div className="bg-white rounded-2xl shadow-2xl p-6 w-full max-w-sm animate-fadeIn">
              <div className="flex justify-center mb-4 text-red-500 bg-red-50 w-16 h-16 rounded-full items-center mx-auto">
                <AlertCircle size={32} />
              </div>
              <h3 className="text-lg font-bold text-center text-gray-800 mb-2">確定刪除此筆資料？</h3>
              <p className="text-sm text-gray-500 text-center mb-6">此動作無法復原，請確認是否執行。</p>
              
              {deleteError && (
                 <div className="bg-red-50 border border-red-200 text-red-600 text-sm p-3 rounded-lg mb-4">
                   <strong>刪除失敗：</strong><br/>
                   請檢查 Firebase Console → Firestore Database → Rules。<br/>
                   錯誤：{deleteError}
                 </div>
              )}

              <div className="flex gap-3">
                <button 
                  onClick={() => setDeleteId(null)}
                  className="flex-1 py-2.5 bg-gray-100 text-gray-700 font-semibold rounded-xl hover:bg-gray-200 transition-colors"
                  disabled={isDeleting}
                >
                  取消
                </button>
                <button 
                  onClick={executeDelete}
                  disabled={isDeleting}
                  className="flex-1 py-2.5 bg-red-600 text-white font-semibold rounded-xl hover:bg-red-700 transition-colors flex items-center justify-center gap-2"
                >
                  {isDeleting ? '刪除中...' : '確認刪除'}
                </button>
              </div>
            </div>
          </div>
        )}

        <div className="max-w-5xl mx-auto">
          <div className="flex justify-between items-center mb-6">
            <h1 className="text-2xl font-bold text-slate-800 flex items-center gap-2">
              <ShieldCheck className="text-slate-600" />
              管理員後台
            </h1>
            <button 
              onClick={() => setView('user')}
              className="flex items-center gap-2 text-sm text-red-600 font-medium px-4 py-2 bg-white rounded-lg shadow-sm hover:bg-red-50 border border-red-100"
            >
              <LogOut size={16} /> 登出
            </button>
          </div>

          <div className="bg-white rounded-2xl shadow-sm border border-gray-200 overflow-hidden">
            <div className="p-4 border-b border-gray-100 bg-gray-50 flex flex-col md:flex-row md:items-center justify-between gap-4">
              <div className="relative flex-1 max-w-md">
                <Search className="absolute left-3 top-1/2 -translate-y-1/2 text-gray-400" size={18} />
                <input 
                  type="text" 
                  placeholder="搜尋名稱或代碼..." 
                  value={searchTerm}
                  onChange={(e) => setSearchTerm(e.target.value)}
                  className="w-full pl-10 pr-4 py-2 rounded-lg border border-gray-200 focus:outline-none focus:ring-2 focus:ring-blue-500"
                />
              </div>
              <div className="text-sm text-gray-500">
                共 {verifications.length} 筆資料
              </div>
            </div>

            <div className="overflow-x-auto">
              <table className="w-full text-left border-collapse">
                <thead>
                  <tr className="bg-gray-50 text-gray-600 text-sm border-b border-gray-200">
                    <th className="p-4 font-semibold">時間</th>
                    <th className="p-4 font-semibold">應用程式</th>
                    <th className="p-4 font-semibold">用戶資料</th>
                    <th className="p-4 font-semibold">驗證碼</th>
                    <th className="p-4 font-semibold text-right">操作</th>
                  </tr>
                </thead>
                <tbody className="divide-y divide-gray-100">
                  {filteredVerifications.map((v) => {
                    const app = apps.find(a => a.id === v.app);
                    return (
                      <tr key={v.id} className="hover:bg-gray-50 transition-colors">
                        <td className="p-4 text-sm text-gray-500 whitespace-nowrap">
                          {v.createdAt || new Date(v.timestamp).toLocaleString()}
                        </td>
                        <td className="p-4">
                          <span className={`inline-flex items-center gap-1.5 px-2.5 py-1 rounded-full text-xs font-medium bg-white border shadow-sm ${app?.textColor}`}>
                            {app?.icon}
                            {app?.name}
                          </span>
                        </td>
                        <td className="p-4">
                          <div className="font-medium text-gray-900">{v.username}</div>
                          <div className="text-xs text-gray-500 space-y-0.5">
                            {v.phone && <div className="flex items-center gap-1"><Phone size={10}/> {v.phone}</div>}
                            {v.email && <div className="flex items-center gap-1"><Mail size={10}/> {v.email}</div>}
                          </div>
                        </td>
                        <td className="p-4">
                          <span className="font-mono font-bold text-slate-700 bg-slate-100 px-2 py-1 rounded border border-slate-200">
                            {v.code}
                          </span>
                        </td>
                        <td className="p-4 text-right">
                          <button 
                            onClick={() => confirmDelete(v.id)}
                            className="text-gray-400 hover:text-red-500 transition-colors p-2 rounded-lg hover:bg-red-50"
                            title="刪除"
                          >
                            <Trash2 size={18} />
                          </button>
                        </td>
                      </tr>
                    );
                  })}
                  {filteredVerifications.length === 0 && (
                    <tr>
                      <td colSpan="5" className="p-8 text-center text-gray-400">
                        目前沒有相符的驗證資料
                      </td>
                    </tr>
                  )}
                </tbody>
              </table>
            </div>
          </div>
        </div>
        <style>{`
          .animate-fadeIn {
            animation: fadeIn 0.3s ease-out forwards;
          }
          @keyframes fadeIn {
            from { opacity: 0; transform: scale(0.95); }
            to { opacity: 1; transform: scale(1); }
          }
        `}</style>
      </div>
    );
  }

  // --- View: User Form (Default) ---
  return (
    <div className="min-h-screen bg-gray-50 flex items-center justify-center p-4 relative">
      <div className="max-w-md w-full bg-white rounded-2xl shadow-xl overflow-hidden border border-gray-100">
        
        {/* Header */}
        <div className="bg-slate-800 p-6 text-center">
          <div className="mx-auto bg-slate-700 w-12 h-12 rounded-full flex items-center justify-center mb-3 text-white">
            <ShieldCheck size={28} />
          </div>
          <h1 className="text-2xl font-bold text-white tracking-wide">新用戶驗證</h1>
          <p className="text-slate-400 text-sm mt-1">請完成以下步驟以獲取驗證代碼</p>
        </div>

        {/* Content */}
        <div className="p-6">
          {step === 1 ? (
            <div className="space-y-6">
              {/* Step 1: Select App */}
              <div>
                <label className="block text-sm font-semibold text-gray-700 mb-3 flex items-center gap-2">
                  <span className="w-6 h-6 rounded-full bg-slate-100 text-slate-600 flex items-center justify-center text-xs">1</span>
                  選擇您使用的應用程式
                </label>
                <div className="grid grid-cols-1 gap-3">
                  {apps.map((app) => (
                    <button
                      key={app.id}
                      onClick={() => {
                        setFormData({ ...formData, app: app.id, phone: '', email: '' });
                        setError('');
                      }}
                      className={`relative flex items-center p-3 rounded-xl border-2 transition-all duration-200 ${
                        formData.app === app.id
                          ? `border-${app.color.replace('bg-', '')} bg-gray-50 ring-1 ring-${app.color.replace('bg-', '')}`
                          : 'border-gray-100 hover:border-gray-300 hover:bg-gray-50'
                      }`}
                    >
                      <div className={`p-2 rounded-lg ${app.color} text-white mr-3 shadow-sm`}>
                        {app.icon}
                      </div>
                      <span className="font-medium text-gray-700">{app.name}</span>
                      {formData.app === app.id && (
                        <div className={`absolute right-3 w-5 h-5 rounded-full ${app.color} text-white flex items-center justify-center`}>
                          <Check size={14} />
                        </div>
                      )}
                    </button>
                  ))}
                </div>
              </div>

              {/* Step 2: Enter Details */}
              <div className={`transition-all duration-300 ${formData.app ? 'opacity-100' : 'opacity-50 pointer-events-none grayscale'}`}>
                <label className="block text-sm font-semibold text-gray-700 mb-3 flex items-center gap-2">
                  <span className="w-6 h-6 rounded-full bg-slate-100 text-slate-600 flex items-center justify-center text-xs">2</span>
                  填寫驗證資料
                </label>
                
                <div className="space-y-4">
                  <div className="relative">
                    <div className="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none text-gray-400">
                      <User size={18} />
                    </div>
                    <input
                      type="text"
                      value={formData.username}
                      onChange={(e) => {
                        setFormData({ ...formData, username: e.target.value });
                        setError('');
                      }}
                      placeholder={`輸入您的${selectedAppObj?.usernameLabel || '暱稱'}`}
                      disabled={!formData.app}
                      className="w-full pl-10 pr-4 py-3 bg-gray-50 border border-gray-200 rounded-xl focus:ring-2 focus:ring-slate-500 focus:border-slate-500 outline-none transition-all"
                    />
                  </div>

                  {selectedAppObj?.fields.includes('phone') && (
                    <div className="relative animate-fadeIn">
                      <div className="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none text-gray-400">
                        <Phone size={18} />
                      </div>
                      <input
                        type="tel"
                        value={formData.phone}
                        onChange={(e) => {
                          setFormData({ ...formData, phone: e.target.value });
                          setError('');
                        }}
                        placeholder="輸入綁定號碼 (例如: 0912345678)"
                        className="w-full pl-10 pr-4 py-3 bg-gray-50 border border-gray-200 rounded-xl focus:ring-2 focus:ring-slate-500 focus:border-slate-500 outline-none transition-all"
                      />
                    </div>
                  )}

                  {selectedAppObj?.fields.includes('email') && (
                    <div className="relative animate-fadeIn">
                      <div className="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none text-gray-400">
                        <Mail size={18} />
                      </div>
                      <input
                        type="email"
                        value={formData.email}
                        onChange={(e) => {
                          setFormData({ ...formData, email: e.target.value });
                          setError('');
                        }}
                        placeholder="輸入電子郵件 (例如: user@gmail.com)"
                        className="w-full pl-10 pr-4 py-3 bg-gray-50 border border-gray-200 rounded-xl focus:ring-2 focus:ring-slate-500 focus:border-slate-500 outline-none transition-all"
                      />
                    </div>
                  )}
                </div>

                {formData.app && (
                  <p className="mt-3 text-xs text-amber-600 flex items-start gap-1 bg-amber-50 p-2 rounded-lg border border-amber-100">
                    <MessageSquare size={14} className="mt-0.5 shrink-0" />
                    請務必輸入您在 {selectedAppObj?.name} 上的正確資料，以便管理員核對。
                  </p>
                )}
              </div>

              {error && (
                <div className="text-red-500 text-sm font-medium text-center bg-red-50 p-2 rounded-lg animate-pulse">
                  {error}
                </div>
              )}

              <button
                onClick={generateCode}
                disabled={isSubmitting}
                className="w-full bg-slate-800 hover:bg-slate-900 text-white font-bold py-3.5 rounded-xl shadow-lg shadow-slate-200 transition-all active:scale-[0.98] flex items-center justify-center gap-2 disabled:opacity-70 disabled:cursor-not-allowed"
              >
                {isSubmitting ? '處理中...' : '產生驗證碼'}
              </button>
            </div>
          ) : (
            <div className="text-center space-y-6 animate-fadeIn">
              <div className="bg-green-50 rounded-2xl p-6 border border-green-100">
                <div className="w-16 h-16 bg-green-100 rounded-full flex items-center justify-center mx-auto mb-4 text-green-600">
                  <Check size={32} />
                </div>
                <h3 className="text-xl font-bold text-gray-800">驗證碼已生成！</h3>
                <p className="text-sm text-gray-500 mt-1">系統已紀錄您的驗證請求</p>
              </div>

              <div className="space-y-2">
                <div className="text-xs text-gray-400 uppercase tracking-wider font-semibold">您的驗證碼</div>
                <div className="bg-slate-800 text-white text-xl md:text-2xl font-mono font-bold py-4 px-4 break-all rounded-xl tracking-widest shadow-inner relative overflow-hidden group">
                  {verificationCode}
                  <div className="absolute top-0 -left-[100%] w-1/2 h-full bg-gradient-to-r from-transparent via-white/10 to-transparent skew-x-[-25deg] group-hover:animate-shine" />
                </div>
              </div>

              <div className="bg-gray-50 p-4 rounded-xl border border-gray-100 text-left">
                <h4 className="text-sm font-semibold text-gray-700 mb-2">下一步：</h4>
                <ul className="text-sm text-gray-600 space-y-2">
                  <li className="flex gap-2">
                    <span className="font-bold text-slate-400">1.</span>
                    點擊下方按鈕複製完整資料。
                  </li>
                  <li className="flex gap-2">
                    <span className="font-bold text-slate-400">2.</span>
                    前往 <span className={`font-bold ${selectedAppObj?.textColor}`}>{selectedAppObj?.name}</span>。
                  </li>
                  <li className="flex gap-2">
                    <span className="font-bold text-slate-400">3.</span>
                    將資料貼上傳送至聊天室。
                  </li>
                </ul>
              </div>

              <div className="flex gap-3">
                <button
                  onClick={handleReset}
                  className="flex-1 py-3 px-4 border border-gray-200 text-gray-600 font-semibold rounded-xl hover:bg-gray-50 transition-colors flex items-center justify-center gap-2"
                >
                  <RefreshCw size={18} />
                  重新填寫
                </button>
                <button
                  onClick={handleCopy}
                  className={`flex-[2] py-3 px-4 rounded-xl font-bold shadow-md transition-all flex items-center justify-center gap-2 text-white ${
                    copied ? 'bg-green-500 hover:bg-green-600' : 'bg-slate-800 hover:bg-slate-900'
                  }`}
                >
                  {copied ? <Check size={18} /> : <Copy size={18} />}
                  {copied ? '已複製！' : '複製代碼'}
                </button>
              </div>
            </div>
          )}
        </div>
      </div>
      
      {/* Admin Link - Modified for better visibility */}
      <div className="fixed bottom-6 right-6 z-50 opacity-80 hover:opacity-100 transition-opacity">
        <button 
          onClick={() => setView('login')}
          className="bg-white text-slate-400 hover:text-slate-700 p-3 rounded-full shadow-lg border border-slate-100 transition-all hover:scale-110"
          title="管理員入口"
        >
          <Lock size={20} />
        </button>
      </div>

      <style>{`
        @keyframes shine {
          0% { left: -100%; }
          100% { left: 200%; }
        }
        .animate-shine {
          animation: shine 1.5s infinite;
        }
        .animate-fadeIn {
          animation: fadeIn 0.4s ease-out forwards;
        }
        @keyframes fadeIn {
          from { opacity: 0; transform: translateY(10px); }
          to { opacity: 1; transform: translateY(0); }
        }
      `}</style>
    </div>
  );
}
