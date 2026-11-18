# TrackOutimport React, { useState, useEffect, useMemo } from 'react';
import {
  LayoutDashboard,
  Wallet,
  ShoppingBag,
  FileSpreadsheet,
  PieChart as PieChartIcon,
  Target,
  Lightbulb,
  Settings,
  Plus,
  Trash2,
  Edit3,
  TrendingUp,
  TrendingDown,
  Calendar,
  Sparkles,
  CheckCircle2,
  AlertTriangle,
  Sun,
  Moon,
  RefreshCw,
  ArrowUpRight,
  ArrowDownRight,
  Award,
  Check,
  X,
  Menu,
  ChevronRight,
  Info,
  Coins,
  PiggyBank
} from 'lucide-react';
import {
  BarChart,
  Bar,
  XAxis,
  YAxis,
  CartesianGrid,
  Tooltip,
  ResponsiveContainer,
  PieChart,
  Pie,
  Cell,
  LineChart,
  Line,
  Legend,
  AreaChart,
  Area
} from 'recharts';

// ============================================================================
// TIPE DATA & INTERFACES
// ============================================================================

type PageType =
  | 'dashboard'
  | 'income'
  | 'expense'
  | 'recap'
  | 'charts'
  | 'savings'
  | 'tips'
  | 'settings';

interface IncomeItem {
  id: string;
  amount: number;
  source: string;
  date: string; // YYYY-MM-DD
  note: string;
}

interface ExpenseItem {
  id: string;
  name: string;
  amount: number;
  category: string;
  date: string; // YYYY-MM-DD
  note: string;
}

interface SavingsGoal {
  id: string;
  title: string;
  targetAmount: number;
  currentAmount: number;
  deadline: string; // YYYY-MM-DD
  createdAt: string;
}

interface ChallengeItem {
  id: string;
  title: string;
  description: string;
  rewardText: string;
  dailyAmount?: number;
  days: boolean[];
}

interface UserProfile {
  name: string;
  darkMode: boolean;
}

interface ToastNotification {
  id: string;
  type: 'success' | 'warning' | 'error' | 'celebration';
  title: string;
  message: string;
}

// ============================================================================
// KONSTANTA & DATA REFERENSI
// ============================================================================

const INCOME_SOURCES = [
  'Uang saku harian',
  'Uang saku mingguan',
  'Uang saku bulanan',
  'Pemberian orang tua',
  'Hadiah',
  'Hasil usaha kecil',
  'Lainnya'
];

const EXPENSE_CATEGORIES = [
  { label: '🍜 Makanan dan Minuman', short: 'Makanan & Minuman', emoji: '🍜', color: '#0284C7' },
  { label: '🚌 Transportasi', short: 'Transportasi', emoji: '🚌', color: '#38BDF8' },
  { label: '📚 Kebutuhan Sekolah', short: 'Kebutuhan Sekolah', emoji: '📚', color: '#1E3A8A' },
  { label: '🖊️ Alat Tulis', short: 'Alat Tulis', emoji: '🖊️', color: '#6366F1' },
  { label: '🛍️ Belanja', short: 'Belanja', emoji: '🛍️', color: '#EC4899' },
  { label: '🎮 Hiburan', short: 'Hiburan', emoji: '🎮', color: '#8B5CF6' },
  { label: '📱 Pulsa dan Internet', short: 'Pulsa & Internet', emoji: '📱', color: '#14B8A6' },
  { label: '🧴 Kebutuhan Pribadi', short: 'Kebutuhan Pribadi', emoji: '🧴', color: '#F59E0B' },
  { label: '📦 Lainnya', short: 'Lainnya', emoji: '📦', color: '#64748B' }
];

const MOTIVATIONAL_QUOTES = [
  'Sedikit demi sedikit, lama-lama menjadi bukit!',
  'Jangan habiskan uang sakumu hanya untuk jajan.',
  'Belajar hemat hari ini, nikmati hasilnya nanti.',
  'Menabung bukan soal banyaknya uang, tetapi kebiasaan.',
  'Pelajar cerdas tahu bedanya kebutuhan dan keinginan!',
  'Uang saku teratur, masa depan makin makmur!'
];

const HEMAT_TIPS = [
  {
    icon: '🍱',
    title: 'Bawa Bekal & Botol Minum dari Rumah',
    desc: 'Membawa bekal makan siang dan air minum bisa menghemat hingga 50% uang jajan harianmu di kantin sekolah.'
  },
  {
    icon: '💰',
    title: 'Sisihkan Tabungan di Awal, Bukan Sisa',
    desc: 'Begitu menerima uang saku, langsung sisihkan 20% untuk target tabungan sebelum dipakai jajan.'
  },
  {
    icon: '📝',
    title: 'Buat Daftar Kebutuhan Sekolah',
    desc: 'Catat alat tulis atau buku yang benar-benar habis agar tidak lapar mata membeli barang lucu yang tidak terpakai.'
  },
  {
    icon: '⚖️',
    title: 'Bedakan Kebutuhan vs Keinginan',
    desc: 'Tunggu 24 jam sebelum membeli barang non-sekolah. Biasanya keinginan impulsif akan hilang keesokan harinya.'
  },
  {
    icon: '🚫',
    title: 'Hindari Jajan Lapar Mata',
    desc: 'Batasi jajan minuman kekinian maksimal 1-2 kali seminggu. Air putih jauh lebih sehat dan gratis!'
  },
  {
    icon: '📊',
    title: 'Cek Grafik TrackOut Tiap Akhir Pekan',
    desc: 'Evaluasi kategori pengeluaran terbesar setiap hari Minggu agar minggu depannya bisa lebih disiplin.'
  }
];

// ============================================================================
// FUNGSI UTILITAS FORMATTER & TANGGAL
// ============================================================================

const formatRupiah = (amount: number): string => {
  const isNegative = amount < 0;
  const absVal = Math.abs(Math.round(amount));
  const formatted = absVal.toString().replace(/\B(?=(\d{3})+(?!\d))/g, '.');
  return `${isNegative ? '-' : ''}Rp${formatted}`;
};

const getTodayISO = (): string => {
  const now = new Date();
  const offset = now.getTimezoneOffset() * 60000;
  return new Date(now.getTime() - offset).toISOString().split('T')[0];
};

const getOffsetDateISO = (daysOffset: number): string => {
  const d = new Date();
  d.setDate(d.getDate() + daysOffset);
  const offset = d.getTimezoneOffset() * 60000;
  return new Date(d.getTime() - offset).toISOString().split('T')[0];
};

const formatDateIndo = (dateStr: string): string => {
  if (!dateStr) return '-';
  const [year, month, day] = dateStr.split('-').map(Number);
  const dateObj = new Date(year, month - 1, day);
  return dateObj.toLocaleDateString('id-ID', {
    day: 'numeric',
    month: 'short',
    year: 'numeric'
  });
};

const formatFullTodayIndo = (): string => {
  return new Date().toLocaleDateString('id-ID', {
    weekday: 'long',
    day: 'numeric',
    month: 'long',
    year: 'numeric'
  });
};

// ============================================================================
// DATA SIMULASI AWAL (DEFAULT SEED DATA)
// ============================================================================

const createInitialData = () => {
  const today = getTodayISO();
  const yesterday = getOffsetDateISO(-1);
  const twoDaysAgo = getOffsetDateISO(-2);
  const threeDaysAgo = getOffsetDateISO(-3);
  const nextMonth = getOffsetDateISO(30);
  const twoMonthsLater = getOffsetDateISO(60);

  const initialProfile: UserProfile = {
    name: 'Raka Pratama',
    darkMode: false
  };

  const initialIncomes: IncomeItem[] = [
    {
      id: 'inc-1',
      amount: 600000,
      source: 'Uang saku mingguan',
      date: threeDaysAgo,
      note: 'Uang saku dari Ayah & Ibu untuk minggu ini'
    }
  ];

  const initialExpenses: ExpenseItem[] = [
    {
      id: 'exp-1',
      name: 'Makan di kantin',
      amount: 12000,
      category: '🍜 Makanan dan Minuman',
      date: today,
      note: 'Nasi goreng spesial kantin sekolah'
    },
    {
      id: 'exp-2',
      name: 'Es teh',
      amount: 5000,
      category: '🍜 Makanan dan Minuman',
      date: today,
      note: 'Es teh manis dingin istirahat kedua'
    },
    {
      id: 'exp-3',
      name: 'Transportasi',
      amount: 10000,
      category: '🚌 Transportasi',
      date: yesterday,
      note: 'Ongkos angkot & ojek pulang sekolah'
    },
    {
      id: 'exp-4',
      name: 'Buku tulis',
      amount: 8000,
      category: '📚 Kebutuhan Sekolah',
      date: twoDaysAgo,
      note: 'Buku catatan mata pelajaran Fisika'
    },
    {
      id: 'exp-5',
      name: 'Pulpen',
      amount: 4000,
      category: '🖊️ Alat Tulis',
      date: twoDaysAgo,
      note: 'Pulpen gel tinta hitam 0.5mm'
    }
  ];

  const initialSavings: SavingsGoal[] = [
    {
      id: 'sav-1',
      title: 'Membeli Sepatu Sekolah Baru',
      targetAmount: 350000,
      currentAmount: 150000,
      deadline: nextMonth,
      createdAt: threeDaysAgo
    },
    {
      id: 'sav-2',
      title: 'Headset Belajar & Musik',
      targetAmount: 250000,
      currentAmount: 100000,
      deadline: twoMonthsLater,
      createdAt: threeDaysAgo
    }
  ];

  const initialChallenges: ChallengeItem[] = [
    {
      id: 'chal-1',
      title: 'Tantangan Menabung 7 Hari',
      description: 'Sisihkan minimal Rp5.000 setiap hari selama 7 hari berturut-turut!',
      rewardText: 'Terkumpul Rp35.000 ekstra',
      dailyAmount: 5000,
      days: [true, true, true, false, false, false, false]
    },
    {
      id: 'chal-2',
      title: 'Tantangan Kurangi Jajan 5 Hari',
      description: 'Bawa bekal dari rumah & jajan maksimal Rp10.000/hari selama 5 hari sekolah.',
      rewardText: 'Hemat hingga Rp50.000/minggu',
      days: [true, true, false, false, false]
    },
    {
      id: 'chal-3',
      title: 'Tantangan Zero Minuman Kemasan',
      description: 'Bawa botol minum isi ulang dari rumah selama 7 hari penuh.',
      rewardText: 'Sehat & hemat Rp35.000',
      days: [true, false, false, false, false, false, false]
    }
  ];

  return {
    profile: initialProfile,
    incomes: initialIncomes,
    expenses: initialExpenses,
    savings: initialSavings,
    challenges: initialChallenges
  };
};

// ============================================================================
// KOMPONEN LOGO CUSTOM SVG TRACKOUT
// ============================================================================

const TrackOutLogo: React.FC<{ size?: 'sm' | 'md' | 'lg'; lightText?: boolean }> = ({
  size = 'md',
  lightText = false
}) => {
  const iconDimensions = size === 'sm' ? 'w-8 h-8' : size === 'lg' ? 'w-12 h-12' : 'w-10 h-10';
  const titleSize = size === 'sm' ? 'text-lg' : size === 'lg' ? 'text-2xl' : 'text-xl';

  return (
    <div className="flex items-center gap-2.5 select-none">
      <div
        className={`${iconDimensions} rounded-xl bg-gradient-to-br from-[#0F172A] via-[#1E3A8A] to-[#0284C7] p-1.5 shadow-lg shadow-sky-500/20 flex items-center justify-center relative overflow-hidden`}
      >
        <svg viewBox="0 0 40 40" fill="none" className="w-full h-full">
          {/* Bentuk Dompet / Kartu Pelajar Modern */}
          <rect
            x="4"
            y="10"
            width="32"
            height="22"
            rx="6"
            fill="white"
            fillOpacity="0.14"
            stroke="#38BDF8"
            strokeWidth="2"
          />
          {/* Grafik Batang Mini di dalam */}
          <rect x="10" y="21" width="3.5" height="6" rx="1.5" fill="#38BDF8" />
          <rect x="16" y="17" width="3.5" height="10" rx="1.5" fill="#7DD3FC" />
          <rect x="22" y="13" width="3.5" height="14" rx="1.5" fill="#10B981" />
          {/* Panah Naik Impian */}
          <path
            d="M10 18L17 13L22 15L31 7"
            stroke="#FFFFFF"
            strokeWidth="2.4"
            strokeLinecap="round"
            strokeLinejoin="round"
          />
          <path
            d="M26 7H31V12"
            stroke="#38BDF8"
            strokeWidth="2.4"
            strokeLinecap="round"
            strokeLinejoin="round"
          />
        </svg>
      </div>
      <div className="flex flex-col">
        <span
          className={`${titleSize} font-extrabold tracking-tight leading-none ${
            lightText
              ? 'text-white'
              : 'bg-gradient-to-r from-[#0F172A] via-[#1E3A8A] to-[#0284C7] dark:from-white dark:via-sky-200 dark:to-sky-400 bg-clip-text text-transparent'
          }`}
        >
          Track<span className="text-[#0284C7] dark:text-[#38BDF8]">Out</span>
        </span>
        {size !== 'sm' && (
          <span className="text-[10px] font-medium text-slate-500 dark:text-slate-400 tracking-wide mt-0.5">
            Atur Uang Sakumu, Raih Impianmu!
          </span>
        )}
      </div>
    </div>
  );
};

// ============================================================================
// KOMPONEN UTAMA APLIKASI
// ============================================================================

export default function App() {
  // --- STATE PERSISTENCE DENGAN LOCAL STORAGE ---
  const [profile, setProfile] = useState<UserProfile>(() => {
    try {
      const saved = localStorage.getItem('trackout_profile');
      return saved ? JSON.parse(saved) : createInitialData().profile;
    } catch {
      return createInitialData().profile;
    }
  });

  const [incomes, setIncomes] = useState<IncomeItem[]>(() => {
    try {
      const saved = localStorage.getItem('trackout_incomes');
      return saved ? JSON.parse(saved) : createInitialData().incomes;
    } catch {
      return createInitialData().incomes;
    }
  });

  const [expenses, setExpenses] = useState<ExpenseItem[]>(() => {
    try {
      const saved = localStorage.getItem('trackout_expenses');
      return saved ? JSON.parse(saved) : createInitialData().expenses;
    } catch {
      return createInitialData().expenses;
    }
  });

  const [savingsGoals, setSavingsGoals] = useState<SavingsGoal[]>(() => {
    try {
      const saved = localStorage.getItem('trackout_savings');
      return saved ? JSON.parse(saved) : createInitialData().savings;
    } catch {
      return createInitialData().savings;
    }
  });

  const [challenges, setChallenges] = useState<ChallengeItem[]>(() => {
    try {
      const saved = localStorage.getItem('trackout_challenges');
      return saved ? JSON.parse(saved) : createInitialData().challenges;
    } catch {
      return createInitialData().challenges;
    }
  });

  // --- STATE NAVIGASI & UI ---
  const [activePage, setActivePage] = useState<PageType>('dashboard');
  const [mobileMoreOpen, setMobileMoreOpen] = useState(false);
  const [quoteIndex, setQuoteIndex] = useState(0);
  const [toasts, setToasts] = useState<ToastNotification[]>([]);

  // --- STATE FORM UANG SAKU (PEMASUKAN) ---
  const [editingIncomeId, setEditingIncomeId] = useState<string | null>(null);
  const [incAmount, setIncAmount] = useState('');
  const [incSource, setIncSource] = useState(INCOME_SOURCES[0]);
  const [incDate, setIncDate] = useState(getTodayISO());
  const [incNote, setIncNote] = useState('');

  // --- STATE FORM PENGELUARAN ---
  const [editingExpenseId, setEditingExpenseId] = useState<string | null>(null);
  const [expName, setExpName] = useState('');
  const [expAmount, setExpAmount] = useState('');
  const [expCategory, setExpCategory] = useState(EXPENSE_CATEGORIES[0].label);
  const [expDate, setExpDate] = useState(getTodayISO());
  const [expNote, setExpNote] = useState('');
  const [insufficientBalanceAlert, setInsufficientBalanceAlert] = useState<string | null>(null);

  // --- STATE FILTER REKAP KEUANGAN ---
  const [recapFilter, setRecapFilter] = useState<'today' | 'week' | 'month' | 'custom'>('month');
  const [customStartDate, setCustomStartDate] = useState(getOffsetDateISO(-7));
  const [customEndDate, setCustomEndDate] = useState(getTodayISO());

  // --- STATE FORM & MODAL TARGET MENABUNG ---
  const [savTitle, setSavTitle] = useState('');
  const [savTarget, setSavTarget] = useState('');
  const [savInitial, setSavInitial] = useState('');
  const [savDeadline, setSavDeadline] = useState(getOffsetDateISO(30));
  const [depositModalGoal, setDepositModalGoal] = useState<SavingsGoal | null>(null);
  const [depositAmount, setDepositAmount] = useState('');
  const [deductFromPocket, setDeductFromPocket] = useState(true);

  // --- STATE PENGATURAN & KONFIRMASI RESET ---
  const [tempName, setTempName] = useState(profile.name);
  const [showResetConfirm, setShowResetConfirm] = useState(false);

  // --- SINKRONISASI LOCAL STORAGE ---
  useEffect(() => {
    localStorage.setItem('trackout_profile', JSON.stringify(profile));
  }, [profile]);

  useEffect(() => {
    localStorage.setItem('trackout_incomes', JSON.stringify(incomes));
  }, [incomes]);

  useEffect(() => {
    localStorage.setItem('trackout_expenses', JSON.stringify(expenses));
  }, [expenses]);

  useEffect(() => {
    localStorage.setItem('trackout_savings', JSON.stringify(savingsGoals));
  }, [savingsGoals]);

  useEffect(() => {
    localStorage.setItem('trackout_challenges', JSON.stringify(challenges));
  }, [challenges]);

  // Rotasi otomatis kalimat motivasi setiap 12 detik
  useEffect(() => {
    const timer = setInterval(() => {
      setQuoteIndex((prev) => (prev + 1) % MOTIVATIONAL_QUOTES.length);
    }, 12000);
    return () => clearInterval(timer);
  }, []);

  // Helper Notifikasi Toast
  const addToast = (
    type: ToastNotification['type'],
    title: string,
    message: string
  ) => {
    const id = `toast-${Date.now()}-${Math.random()}`;
    setToasts((prev) => [...prev, { id, type, title, message }]);
    setTimeout(() => {
      setToasts((prev) => prev.filter((t) => t.id !== id));
    }, 4500);
  };

  // ============================================================================
  // PERHITUNGAN KEUANGAN INTI (OTOMATIS & AKURAT)
  // ============================================================================

  const totalAllIncome = useMemo(
    () => incomes.reduce((sum, item) => sum + item.amount, 0),
    [incomes]
  );

  const totalAllExpense = useMemo(
    () => expenses.reduce((sum, item) => sum + item.amount, 0),
    [expenses]
  );

  const currentBalance = useMemo(
    () => totalAllIncome - totalAllExpense,
    [totalAllIncome, totalAllExpense]
  );

  const totalSavedAmount = useMemo(
    () => savingsGoals.reduce((sum, item) => sum + item.currentAmount, 0),
    [savingsGoals]
  );

  const totalSavingsTarget = useMemo(
    () => savingsGoals.reduce((sum, item) => sum + item.targetAmount, 0),
    [savingsGoals]
  );

  // Perhitungan Bulan Ini (untuk Dashboard)
  const currentMonthPrefix = getTodayISO().slice(0, 7); // YYYY-MM

  const monthIncome = useMemo(
    () =>
      incomes
        .filter((i) => i.date.startsWith(currentMonthPrefix))
        .reduce((sum, i) => sum + i.amount, 0),
    [incomes, currentMonthPrefix]
  );

  const monthExpense = useMemo(
    () =>
      expenses
        .filter((e) => e.date.startsWith(currentMonthPrefix))
        .reduce((sum, e) => sum + e.amount, 0),
    [expenses, currentMonthPrefix]
  );

  // Transaksi Gabungan Terbaru (untuk Dashboard)
  const recentTransactions = useMemo(() => {
    const combined = [
      ...incomes.map((i) => ({
        id: i.id,
        type: 'income' as const,
        title: i.source,
        subtitle: i.note || 'Pemasukan uang saku',
        amount: i.amount,
        date: i.date
      })),
      ...expenses.map((e) => ({
        id: e.id,
        type: 'expense' as const,
        title: e.name,
        subtitle: e.category,
        amount: e.amount,
        date: e.date
      }))
    ];
    return combined.sort((a, b) => b.date.localeCompare(a.date)).slice(0, 6);
  }, [incomes, expenses]);

  // ============================================================================
  // HANDLER CRUD PEMASUKAN (UANG SAKU)
  // ============================================================================

  const handleSaveIncome = (e: React.FormEvent) => {
    e.preventDefault();
    const numericAmount = Number(incAmount);
    if (!numericAmount || numericAmount <= 0) {
      addToast('warning', 'Nominal Tidak Valid', 'Masukkan nominal uang saku lebih dari Rp0.');
      return;
    }

    if (editingIncomeId) {
      setIncomes((prev) =>
        prev.map((item) =>
          item.id === editingIncomeId
            ? {
                ...item,
                amount: numericAmount,
                source: incSource,
                date: incDate,
                note: incNote.trim()
              }
            : item
        )
      );
      addToast('success', 'Pemasukan Diperbarui', `Data uang saku sebesar ${formatRupiah(numericAmount)} berhasil diubah.`);
      setEditingIncomeId(null);
    } else {
      const newIncome: IncomeItem = {
        id: `inc-${Date.now()}`,
        amount: numericAmount,
        source: incSource,
        date: incDate,
        note: incNote.trim()
      };
      setIncomes((prev) => [newIncome, ...prev]);
      addToast('success', 'Uang Saku Tercatat! 💵', `+${formatRupiah(numericAmount)} ditambahkan ke saldomu.`);
    }

    setIncAmount('');
    setIncSource(INCOME_SOURCES[0]);
    setIncDate(getTodayISO());
    setIncNote('');
  };

  const handleEditIncome = (item: IncomeItem) => {
    setEditingIncomeId(item.id);
    setIncAmount(String(item.amount));
    setIncSource(item.source);
    setIncDate(item.date);
    setIncNote(item.note);
    window.scrollTo({ top: 0, behavior: 'smooth' });
  };

  const handleDeleteIncome = (id: string) => {
    setIncomes((prev) => prev.filter((item) => item.id !== id));
    if (editingIncomeId === id) {
      setEditingIncomeId(null);
      setIncAmount('');
      setIncNote('');
    }
    addToast('info' as any, 'Pemasukan Dihapus', 'Catatan uang saku telah dihapus dan saldo diperbarui.');
  };

  // ============================================================================
  // HANDLER CRUD PENGELUARAN HARIAN
  // ============================================================================

  const handleSaveExpense = (e: React.FormEvent) => {
    e.preventDefault();
    setInsufficientBalanceAlert(null);

    const numericAmount = Number(expAmount);
    if (!expName.trim()) {
      addToast('warning', 'Nama Kosong', 'Isi nama pengeluaranmu dulu ya!');
      return;
    }
    if (!numericAmount || numericAmount <= 0) {
      addToast('warning', 'Nominal Tidak Valid', 'Masukkan nominal pengeluaran lebih dari Rp0.');
      return;
    }

    // Hitung saldo yang tersedia (jika sedang edit, kembalikan dulu nilai lama item yang diedit)
    const oldItem = editingExpenseId
      ? expenses.find((item) => item.id === editingExpenseId)
      : null;
    const availableBalance = currentBalance + (oldItem ? oldItem.amount : 0);

    if (numericAmount > availableBalance) {
      const msg = `Uang saku tidak mencukupi! Sisa uang sakumu saat ini ${formatRupiah(
        availableBalance
      )}, sedangkan pengeluaran yang dicatat ${formatRupiah(numericAmount)}.`;
      setInsufficientBalanceAlert(msg);
      addToast('error', 'Saldo Tidak Cukup! ⚠️', 'Pengeluaran melebihi sisa uang sakumu.');
      return;
    }

    if (editingExpenseId) {
      setExpenses((prev) =>
        prev.map((item) =>
          item.id === editingExpenseId
            ? {
                ...item,
                name: expName.trim(),
                amount: numericAmount,
                category: expCategory,
                date: expDate,
                note: expNote.trim()
              }
            : item
        )
      );
      addToast('success', 'Pengeluaran Diperbarui', `Catatan "${expName.trim()}" berhasil diperbarui.`);
      setEditingExpenseId(null);
    } else {
      const newExpense: ExpenseItem = {
        id: `exp-${Date.now()}`,
        name: expName.trim(),
        amount: numericAmount,
        category: expCategory,
        date: expDate,
        note: expNote.trim()
      };
      setExpenses((prev) => [newExpense, ...prev]);
      addToast('success', 'Pengeluaran Tercatat', `-${formatRupiah(numericAmount)} untuk ${expName.trim()}.`);
    }

    setExpName('');
    setExpAmount('');
    setExpCategory(EXPENSE_CATEGORIES[0].label);
    setExpDate(getTodayISO());
    setExpNote('');
  };

  const handleEditExpense = (item: ExpenseItem) => {
    setInsufficientBalanceAlert(null);
    setEditingExpenseId(item.id);
    setExpName(item.name);
    setExpAmount(String(item.amount));
    setExpCategory(item.category);
    setExpDate(item.date);
    setExpNote(item.note);
    window.scrollTo({ top: 0, behavior: 'smooth' });
  };

  const handleDeleteExpense = (id: string) => {
    setExpenses((prev) => prev.filter((item) => item.id !== id));
    if (editingExpenseId === id) {
      setEditingExpenseId(null);
      setExpName('');
      setExpAmount('');
      setExpNote('');
    }
    addToast('success', 'Pengeluaran Dihapus', 'Catatan pengeluaran dihapus dan saldo dikembalikan.');
  };

  // ============================================================================
  // HANDLER TARGET MENABUNG
  // ============================================================================

  const handleCreateGoal = (e: React.FormEvent) => {
    e.preventDefault();
    const targetNum = Number(savTarget);
    const initialNum = Number(savInitial) || 0;

    if (!savTitle.trim() || !targetNum || targetNum <= 0) {
      addToast('warning', 'Data Belum Lengkap', 'Masukkan nama target dan nominal target yang valid.');
      return;
    }

    const newGoal: SavingsGoal = {
      id: `sav-${Date.now()}`,
      title: savTitle.trim(),
      targetAmount: targetNum,
      currentAmount: initialNum,
      deadline: savDeadline,
      createdAt: getTodayISO()
    };

    setSavingsGoals((prev) => [newGoal, ...prev]);
    setSavTitle('');
    setSavTarget('');
    setSavInitial('');
    setSavDeadline(getOffsetDateISO(30));

    if (initialNum >= targetNum) {
      addToast('celebration', 'Selamat! 🎉', 'Kamu berhasil mencapai target tabunganmu!');
    } else {
      addToast('success', 'Target Tabungan Dibuat! 🎯', `Semangat menabung untuk "${newGoal.title}"!`);
    }
  };

  const handleAddDeposit = (e: React.FormEvent) => {
    e.preventDefault();
    if (!depositModalGoal) return;

    const addVal = Number(depositAmount);
    if (!addVal || addVal <= 0) {
      addToast('warning', 'Nominal Kosong', 'Masukkan nominal tabungan yang ingin disisihkan.');
      return;
    }

    if (deductFromPocket && addVal > currentBalance) {
      addToast(
        'error',
        'Sisa Uang Saku Kurang',
        `Sisa uang sakumu hanya ${formatRupiah(currentBalance)}. Kurangi nominal atau matikan potong saldo.`
      );
      return;
    }

    const updatedAmount = depositModalGoal.currentAmount + addVal;
    const isReached = updatedAmount >= depositModalGoal.targetAmount;

    setSavingsGoals((prev) =>
      prev.map((g) =>
        g.id === depositModalGoal.id ? { ...g, currentAmount: updatedAmount } : g
      )
    );

    if (deductFromPocket) {
      const savingExpense: ExpenseItem = {
        id: `exp-sav-${Date.now()}`,
        name: `Sisihkan Tabungan: ${depositModalGoal.title}`,
        amount: addVal,
        category: '📦 Lainnya',
        date: getTodayISO(),
        note: 'Setoran ke target tabungan TrackOut'
      };
      setExpenses((prev) => [savingExpense, ...prev]);
    }

    setDepositModalGoal(null);
    setDepositAmount('');

    if (isReached) {
      addToast(
        'celebration',
        'Selamat! 🎉 Kamu berhasil mencapai target tabunganmu!',
        `Target "${depositModalGoal.title}" telah terpenuhi 100%!`
      );
    } else {
      addToast(
        'success',
        'Tabungan Bertambah! 💰',
        `Berhasil menyisihkan ${formatRupiah(addVal)} untuk ${depositModalGoal.title}.`
      );
    }
  };

  const handleDeleteGoal = (id: string) => {
    setSavingsGoals((prev) => prev.filter((g) => g.id !== id));
    addToast('success', 'Target Dihapus', 'Target tabungan telah dihapus.');
  };

  // ============================================================================
  // HANDLER TANTANGAN HEMAT & PENGATURAN
  // ============================================================================

  const toggleChallengeDay = (challengeId: string, dayIdx: number) => {
    setChallenges((prev) =>
      prev.map((ch) => {
        if (ch.id !== challengeId) return ch;
        const updatedDays = [...ch.days];
        updatedDays[dayIdx] = !updatedDays[dayIdx];
        const allDone = updatedDays.every(Boolean);
        if (allDone && !ch.days.every(Boolean)) {
          addToast('celebration', 'Tantangan Selesai! 🏆', `Keren! Kamu menuntaskan "${ch.title}"!`);
        }
        return { ...ch, days: updatedDays };
      })
    );
  };

  const handleSaveSettings = (e: React.FormEvent) => {
    e.preventDefault();
    setProfile((prev) => ({
      ...prev,
      name: tempName.trim() || 'Pelajar Hebat'
    }));
    addToast('success', 'Profil Disimpan', 'Perubahan nama berhasil diterapkan.');
  };

  const handleResetSimulation = () => {
    const fresh = createInitialData();
    setProfile(fresh.profile);
    setIncomes(fresh.incomes);
    setExpenses(fresh.expenses);
    setSavingsGoals(fresh.savings);
    setChallenges(fresh.challenges);
    setTempName(fresh.profile.name);
    setShowResetConfirm(false);
    addToast('success', 'Data Direset', 'Semua data simulasi telah dikembalikan ke contoh awal.');
  };

  // ============================================================================
  // DATA FILTER UNTUK HALAMAN REKAP KEUANGAN
  // ============================================================================

  const filteredRecapData = useMemo(() => {
    const today = getTodayISO();
    const weekAgo = getOffsetDateISO(-7);
    const monthPrefix = today.slice(0, 7);

    const matchDate = (dateStr: string) => {
      if (recapFilter === 'today') return dateStr === today;
      if (recapFilter === 'week') return dateStr >= weekAgo && dateStr <= today;
      if (recapFilter === 'month') return dateStr.startsWith(monthPrefix);
      return dateStr >= customStartDate && dateStr <= customEndDate;
    };

    const filteredInc = incomes.filter((i) => matchDate(i.date));
    const filteredExp = expenses.filter((e) => matchDate(e.date));

    const sumInc = filteredInc.reduce((s, i) => s + i.amount, 0);
    const sumExp = filteredExp.reduce((s, e) => s + e.amount, 0);

    // Rekap Harian Pengeluaran
    const expByDateMap: Record<string, number> = {};
    filteredExp.forEach((e) => {
      expByDateMap[e.date] = (expByDateMap[e.date] || 0) + e.amount;
    });
    const dailyExpenseSummary = Object.entries(expByDateMap)
      .map(([date, total]) => ({ date, total }))
      .sort((a, b) => b.date.localeCompare(a.date));

    // Rekap Harian Pemasukan
    const incByDateMap: Record<string, number> = {};
    filteredInc.forEach((i) => {
      incByDateMap[i.date] = (incByDateMap[i.date] || 0) + i.amount;
    });
    const dailyIncomeSummary = Object.entries(incByDateMap)
      .map(([date, total]) => ({ date, total }))
      .sort((a, b) => b.date.localeCompare(a.date));

    // Riwayat gabungan
    const history = [
      ...filteredInc.map((i) => ({
        id: i.id,
        type: 'income' as const,
        title: i.source,
        categoryOrNote: i.note || 'Pemasukan',
        amount: i.amount,
        date: i.date
      })),
      ...filteredExp.map((e) => ({
        id: e.id,
        type: 'expense' as const,
        title: e.name,
        categoryOrNote: e.category,
        amount: e.amount,
        date: e.date
      }))
    ].sort((a, b) => b.date.localeCompare(a.date));

    return {
      sumInc,
      sumExp,
      netBalance: sumInc - sumExp,
      dailyExpenseSummary,
      dailyIncomeSummary,
      history
    };
  }, [incomes, expenses, recapFilter, customStartDate, customEndDate]);

  // ============================================================================
  // DATA DINAMIS UNTUK GRAFIK (RECHARTS)
  // ============================================================================

  const chartAnalytics = useMemo(() => {
    // 1. Pengeluaran Harian (7 hari terakhir)
    const dailyBarData = [];
    for (let i = 6; i >= 0; i--) {
      const dStr = getOffsetDateISO(-i);
      const total = expenses
        .filter((e) => e.date === dStr)
        .reduce((s, e) => s + e.amount, 0);
      const [, m, d] = dStr.split('-');
      dailyBarData.push({
        tanggal: `${d}/${m}`,
        fullDate: formatDateIndo(dStr),
        Pengeluaran: total
      });
    }

    // 2. Pengeluaran per Kategori (Pie Chart)
    const categoryTotals: Record<string, number> = {};
    expenses.forEach((e) => {
      categoryTotals[e.category] = (categoryTotals[e.category] || 0) + e.amount;
    });

    const pieData = Object.entries(categoryTotals)
      .map(([catLabel, value]) => {
        const meta = EXPENSE_CATEGORIES.find((c) => c.label === catLabel);
        return {
          name: meta ? meta.short : catLabel,
          fullLabel: catLabel,
          value,
          color: meta ? meta.color : '#0284C7'
        };
      })
      .sort((a, b) => b.value - a.value);

    const topCategory = pieData.length > 0 ? pieData[0] : null;

    // Total uang untuk jajan (Makanan dan Minuman)
    const snackTotal = expenses
      .filter((e) => e.category.includes('Makanan dan Minuman'))
      .reduce((s, e) => s + e.amount, 0);

    // 3. Tren Pengeluaran Bulanan (Line Chart - 6 bulan terakhir)
    const monthlyLineData = [];
    const now = new Date();
    for (let i = 5; i >= 0; i--) {
      const d = new Date(now.getFullYear(), now.getMonth() - i, 1);
      const ym = `${d.getFullYear()}-${String(d.getMonth() + 1).padStart(2, '0')}`;
      const monthName = d.toLocaleDateString('id-ID', { month: 'short' });
      const expSum = expenses
        .filter((e) => e.date.startsWith(ym))
        .reduce((s, e) => s + e.amount, 0);
      const incSum = incomes
        .filter((inc) => inc.date.startsWith(ym))
        .reduce((s, inc) => s + inc.amount, 0);

      monthlyLineData.push({
        bulan: monthName,
        Pengeluaran: expSum,
        Pemasukan: incSum
      });
    }

    // 4. Perbandingan Pemasukan vs Pengeluaran Total
    const comparisonData = [
      {
        name: 'Ringkasan Keuangan',
        Pemasukan: totalAllIncome,
        Pengeluaran: totalAllExpense
      }
    ];

    return {
      dailyBarData,
      pieData,
      topCategory,
      snackTotal,
      monthlyLineData,
      comparisonData
    };
  }, [expenses, incomes, totalAllIncome, totalAllExpense]);

  // ============================================================================
  // ITEM MENU NAVIGASI
  // ============================================================================

  const navItems: { id: PageType; label: string; shortLabel: string; icon: React.ReactNode }[] = [
    { id: 'dashboard', label: 'Dashboard', shortLabel: 'Beranda', icon: <LayoutDashboard className="w-5 h-5" /> },
    { id: 'income', label: 'Uang Saku', shortLabel: 'Pemasukan', icon: <Wallet className="w-5 h-5" /> },
    { id: 'expense', label: 'Pengeluaran', shortLabel: 'Jajan', icon: <ShoppingBag className="w-5 h-5" /> },
    { id: 'recap', label: 'Rekap Keuangan', shortLabel: 'Rekap', icon: <FileSpreadsheet className="w-5 h-5" /> },
    { id: 'charts', label: 'Grafik', shortLabel: 'Grafik', icon: <PieChartIcon className="w-5 h-5" /> },
    { id: 'savings', label: 'Target Menabung', shortLabel: 'Target', icon: <Target className="w-5 h-5" /> },
    { id: 'tips', label: 'Tips Hemat', shortLabel: 'Tips', icon: <Lightbulb className="w-5 h-5" /> },
    { id: 'settings', label: 'Pengaturan', shortLabel: 'Setelan', icon: <Settings className="w-5 h-5" /> }
  ];

  // ============================================================================
  // RENDER UI UTAMA
  // ============================================================================

  return (
    <div className={profile.darkMode ? 'dark' : ''}>
      <div className="min-h-screen bg-slate-50 dark:bg-[#0B1120] text-slate-800 dark:text-slate-100 font-sans transition-colors duration-300 flex flex-col lg:flex-row">
        {/* ================= NOTIFIKASI TOAST MENGAMBANG ================= */}
        <div className="fixed top-4 right-4 z-50 flex flex-col gap-2.5 max-w-sm w-[calc(100%-2rem)] pointer-events-none">
          {toasts.map((t) => (
            <div
              key={t.id}
              className={`pointer-events-auto p-4 rounded-2xl shadow-xl border backdrop-blur-md transition-all duration-300 flex items-start gap-3 ${
                t.type === 'celebration'
                  ? 'bg-gradient-to-r from-[#0F172A] to-[#1E3A8A] text-white border-sky-400'
                  : t.type === 'error'
                  ? 'bg-red-50/95 dark:bg-red-950/90 text-red-900 dark:text-red-100 border-red-300 dark:border-red-800'
                  : t.type === 'warning'
                  ? 'bg-amber-50/95 dark:bg-amber-950/90 text-amber-900 dark:text-amber-100 border-amber-300 dark:border-amber-800'
                  : 'bg-white/95 dark:bg-slate-900/95 text-slate-800 dark:text-slate-100 border-sky-200 dark:border-slate-700'
              }`}
            >
              <div className="mt-0.5">
                {t.type === 'celebration' && <Award className="w-5 h-5 text-amber-400" />}
                {t.type === 'error' && <AlertTriangle className="w-5 h-5 text-red-500" />}
                {t.type === 'warning' && <AlertTriangle className="w-5 h-5 text-amber-500" />}
                {t.type === 'success' && <CheckCircle2 className="w-5 h-5 text-emerald-500" />}
              </div>
              <div className="flex-1 text-sm">
                <p className="font-bold">{t.title}</p>
                <p className="opacity-90 text-xs mt-0.5 leading-relaxed">{t.message}</p>
              </div>
            </div>
          ))}
        </div>

        {/* ================= SIDEBAR DESKTOP & TABLET BESAR ================= */}
        <aside className="hidden lg:flex lg:flex-col lg:w-72 lg:fixed lg:inset-y-0 bg-white dark:bg-[#0F172A] border-r border-slate-200/80 dark:border-slate-800 px-5 py-6 z-30">
          <div className="mb-8 px-1">
            <TrackOutLogo size="md" />
          </div>

          {/* Kartu Mini Saldo di Sidebar */}
          <div className="mb-6 p-4 rounded-2xl bg-gradient-to-br from-[#0F172A] via-[#1E3A8A] to-[#0284C7] text-white shadow-lg shadow-blue-900/20">
            <div className="flex items-center justify-between text-xs text-sky-200 mb-1">
              <span>Sisa Uang Sakumu</span>
              <Coins className="w-4 h-4 text-sky-300" />
            </div>
            <div className="text-xl font-extrabold tracking-tight">
              {formatRupiah(currentBalance)}
            </div>
            <div className="mt-2.5 pt-2.5 border-t border-white/15 flex items-center justify-between text-[11px] text-sky-100">
              <span>{profile.name}</span>
              <span className="px-2 py-0.5 rounded-full bg-white/15 font-medium">
                Pelajar
              </span>
            </div>
          </div>

          {/* Menu Navigasi */}
          <nav className="flex-1 space-y-1.5 overflow-y-auto pr-1">
            {navItems.map((item) => {
              const isActive = activePage === item.id;
              return (
                <button
                  key={item.id}
                  onClick={() => setActivePage(item.id)}
                  className={`w-full flex items-center gap-3 px-4 py-3 rounded-xl text-sm font-semibold transition-all duration-200 ${
                    isActive
                      ? 'bg-gradient-to-r from-[#1E3A8A] to-[#0284C7] text-white shadow-md shadow-sky-500/20'
                      : 'text-slate-600 dark:text-slate-300 hover:bg-sky-50 dark:hover:bg-slate-800/70 hover:text-[#1E3A8A] dark:hover:text-sky-300'
                  }`}
                >
                  {item.icon}
                  <span>{item.label}</span>
                </button>
              );
            })}
          </nav>

          {/* Toggle Tema Cepat di Bawah Sidebar */}
          <div className="pt-4 mt-4 border-t border-slate-200 dark:border-slate-800 flex items-center justify-between">
            <span className="text-xs font-medium text-slate-500 dark:text-slate-400">
              Tampilan Layar
            </span>
            <button
              onClick={() => setProfile((p) => ({ ...p, darkMode: !p.darkMode }))}
              className="flex items-center gap-2 px-3 py-1.5 rounded-xl bg-slate-100 dark:bg-slate-800 text-xs font-semibold text-slate-700 dark:text-slate-200 hover:bg-sky-100 dark:hover:bg-slate-700 transition"
            >
              {profile.darkMode ? (
                <>
                  <Sun className="w-4 h-4 text-amber-400" /> Terang
                </>
              ) : (
                <>
                  <Moon className="w-4 h-4 text-[#1E3A8A]" /> Gelap
                </>
              )}
            </button>
          </div>
        </aside>

        {/* ================= HEADER MOBILE / SMARTPHONE ================= */}
        <header className="lg:hidden sticky top-0 z-30 bg-white/90 dark:bg-[#0F172A]/90 backdrop-blur-md border-b border-slate-200 dark:border-slate-800 px-4 py-3 flex items-center justify-between">
          <TrackOutLogo size="sm" />
          <div className="flex items-center gap-2">
            <div className="text-right mr-1">
              <p className="text-[10px] text-slate-500 dark:text-slate-400">Sisa Saldo</p>
              <p className="text-xs font-extrabold text-[#1E3A8A] dark:text-sky-400">
                {formatRupiah(currentBalance)}
              </p>
            </div>
            <button
              onClick={() => setProfile((p) => ({ ...p, darkMode: !p.darkMode }))}
              aria-label="Ubah Tema"
              className="p-2 rounded-xl bg-slate-100 dark:bg-slate-800 text-slate-700 dark:text-slate-200"
            >
              {profile.darkMode ? <Sun className="w-4 h-4 text-amber-400" /> : <Moon className="w-4 h-4" />}
            </button>
          </div>
        </header>

        {/* ================= KONTEN UTAMA ================= */}
        <main className="flex-1 lg:pl-72 pb-24 lg:pb-10">
          <div className="max-w-6xl mx-auto px-4 sm:px-6 lg:px-8 py-6 sm:py-8">
            {/* =================================================================
                HALAMAN 1: DASHBOARD
            ================================================================== */}
            {activePage === 'dashboard' && (
              <div className="space-y-6 animate-fadeIn">
                {/* Banner Sapaan & Motivasi */}
                <div className="rounded-3xl bg-gradient-to-br from-[#0F172A] via-[#1E3A8A] to-[#0284C7] p-6 sm:p-8 text-white shadow-xl shadow-blue-950/15 relative overflow-hidden">
                  <div className="absolute -right-10 -top-10 w-48 h-48 rounded-full bg-sky-400/15 blur-2xl pointer-events-none" />
                  <div className="absolute right-12 bottom-0 w-32 h-32 rounded-full bg-emerald-400/10 blur-xl pointer-events-none" />

                  <div className="flex flex-col md:flex-row md:items-center md:justify-between gap-4 relative z-10">
                    <div>
                      <div className="inline-flex items-center gap-2 px-3 py-1 rounded-full bg-white/10 text-sky-200 text-xs font-medium mb-3">
                        <Calendar className="w-3.5 h-3.5" />
                        <span>{formatFullTodayIndo()}</span>
                      </div>
                      <h1 className="text-2xl sm:text-3xl font-extrabold tracking-tight">
                        Halo, {profile.name}! 👋
                      </h1>
                      <p className="text-sky-100 text-sm mt-1">
                        Yuk pantau uang sakumu hari ini agar makin hemat dan disiplin!
                      </p>
                    </div>

                    {/* Aksi Cepat */}
                    <div className="flex flex-wrap gap-2.5">
                      <button
                        onClick={() => setActivePage('income')}
                        className="flex items-center gap-1.5 px-4 py-2.5 rounded-xl bg-emerald-500 hover:bg-emerald-600 text-white text-xs sm:text-sm font-bold shadow-lg shadow-emerald-950/20 transition"
                      >
                        <Plus className="w-4 h-4" /> Uang Saku
                      </button>
                      <button
                        onClick={() => setActivePage('expense')}
                        className="flex items-center gap-1.5 px-4 py-2.5 rounded-xl bg-white text-[#0F172A] hover:bg-sky-50 text-xs sm:text-sm font-bold shadow-lg transition"
                      >
                        <Plus className="w-4 h-4 text-red-500" /> Pengeluaran
                      </button>
                    </div>
                  </div>

                  {/* Kotak Kalimat Motivasi Dinamis */}
                  <div className="mt-6 p-4 rounded-2xl bg-white/10 backdrop-blur-sm border border-white/15 flex items-center justify-between gap-3">
                    <div className="flex items-center gap-3">
                      <div className="p-2 rounded-xl bg-amber-400/20 text-amber-300 shrink-0">
                        <Sparkles className="w-5 h-5" />
                      </div>
                      <p className="text-xs sm:text-sm font-medium text-white italic">
                        "{MOTIVATIONAL_QUOTES[quoteIndex]}"
                      </p>
                    </div>
                    <button
                      onClick={() => setQuoteIndex((prev) => (prev + 1) % MOTIVATIONAL_QUOTES.length)}
                      title="Ganti kutipan motivasi"
                      className="p-2 rounded-xl bg-white/10 hover:bg-white/20 text-sky-200 transition shrink-0"
                    >
                      <RefreshCw className="w-4 h-4" />
                    </button>
                  </div>
                </div>

                {/* 4 Kartu Ringkasan Utama */}
                <div className="grid grid-cols-1 sm:grid-cols-2 xl:grid-cols-4 gap-4">
                  {/* Sisa Uang Saku */}
                  <div className="p-5 rounded-2xl bg-white dark:bg-[#0F172A] border border-slate-200/80 dark:border-slate-800 shadow-sm hover:shadow-md transition">
                    <div className="flex items-center justify-between mb-3">
                      <span className="text-xs font-semibold text-slate-500 dark:text-slate-400">
                        Sisa Uang Saku
                      </span>
                      <div className="p-2.5 rounded-xl bg-sky-100 dark:bg-sky-900/40 text-[#0284C7] dark:text-sky-400">
                        <Wallet className="w-5 h-5" />
                      </div>
                    </div>
                    <p className="text-2xl font-extrabold text-[#0F172A] dark:text-white">
                      {formatRupiah(currentBalance)}
                    </p>
                    <p className="text-xs text-sky-600 dark:text-sky-400 mt-1.5 font-medium">
                      Saldo siap pakai saat ini
                    </p>
                  </div>

                  {/* Total Uang Saku Bulan Ini */}
                  <div className="p-5 rounded-2xl bg-white dark:bg-[#0F172A] border border-slate-200/80 dark:border-slate-800 shadow-sm hover:shadow-md transition">
                    <div className="flex items-center justify-between mb-3">
                      <span className="text-xs font-semibold text-slate-500 dark:text-slate-400">
                        Uang Saku Bulan Ini
                      </span>
                      <div className="p-2.5 rounded-xl bg-emerald-100 dark:bg-emerald-900/40 text-emerald-600 dark:text-emerald-400">
                        <TrendingUp className="w-5 h-5" />
                      </div>
                    </div>
                    <p className="text-2xl font-extrabold text-emerald-600 dark:text-emerald-400">
                      {formatRupiah(monthIncome)}
                    </p>
                    <p className="text-xs text-slate-500 dark:text-slate-400 mt-1.5">
                      Total pemasukan bulan ini
                    </p>
                  </div>

                  {/* Total Pengeluaran Bulan Ini */}
                  <div className="p-5 rounded-2xl bg-white dark:bg-[#0F172A] border border-slate-200/80 dark:border-slate-800 shadow-sm hover:shadow-md transition">
                    <div className="flex items-center justify-between mb-3">
                      <span className="text-xs font-semibold text-slate-500 dark:text-slate-400">
                        Pengeluaran Bulan Ini
                      </span>
                      <div className="p-2.5 rounded-xl bg-red-100 dark:bg-red-900/40 text-red-600 dark:text-red-400">
                        <TrendingDown className="w-5 h-5" />
                      </div>
                    </div>
                    <p className="text-2xl font-extrabold text-red-600 dark:text-red-400">
                      {formatRupiah(monthExpense)}
                    </p>
                    <p className="text-xs text-slate-500 dark:text-slate-400 mt-1.5">
                      Total jajan & kebutuhan bulan ini
                    </p>
                  </div>

                  {/* Target Tabungan */}
                  <div className="p-5 rounded-2xl bg-white dark:bg-[#0F172A] border border-slate-200/80 dark:border-slate-800 shadow-sm hover:shadow-md transition">
                    <div className="flex items-center justify-between mb-3">
                      <span className="text-xs font-semibold text-slate-500 dark:text-slate-400">
                        Total Tabungan Impian
                      </span>
                      <div className="p-2.5 rounded-xl bg-indigo-100 dark:bg-indigo-900/40 text-[#1E3A8A] dark:text-indigo-400">
                        <PiggyBank className="w-5 h-5" />
                      </div>
                    </div>
                    <p className="text-2xl font-extrabold text-[#1E3A8A] dark:text-sky-300">
                      {formatRupiah(totalSavedAmount)}
                    </p>
                    <p className="text-xs text-slate-500 dark:text-slate-400 mt-1.5">
                      Dari total target {formatRupiah(totalSavingsTarget)}
                    </p>
                  </div>
                </div>

                {/* Grid Bawah Dashboard: Transaksi Terakhir & Progress Target Tabungan */}
                <div className="grid grid-cols-1 lg:grid-cols-12 gap-6">
                  {/* Transaksi Terakhir */}
                  <div className="lg:col-span-7 p-6 rounded-2xl bg-white dark:bg-[#0F172A] border border-slate-200/80 dark:border-slate-800 shadow-sm">
                    <div className="flex items-center justify-between mb-5">
                      <div>
                        <h2 className="text-lg font-bold text-slate-900 dark:text-white">
                          Transaksi Terakhir
                        </h2>
                        <p className="text-xs text-slate-500 dark:text-slate-400">
                          Catatan uang saku dan pengeluaran terbarumu
                        </p>
                      </div>
                      <button
                        onClick={() => setActivePage('recap')}
                        className="text-xs font-bold text-[#0284C7] dark:text-sky-400 hover:underline flex items-center gap-1"
                      >
                        Lihat Semua <ChevronRight className="w-4 h-4" />
                      </button>
                    </div>

                    {recentTransactions.length === 0 ? (
                      <div className="text-center py-10 text-slate-400 text-sm">
                        Belum ada transaksi tercatat. Yuk mulai catat!
                      </div>
                    ) : (
                      <div className="space-y-3">
                        {recentTransactions.map((tx) => (
                          <div
                            key={tx.id}
                            className="flex items-center justify-between p-3.5 rounded-xl bg-slate-50 dark:bg-slate-800/60 hover:bg-sky-50/70 dark:hover:bg-slate-800 transition"
                          >
                            <div className="flex items-center gap-3 min-w-0">
                              <div
                                className={`p-2.5 rounded-xl shrink-0 ${
                                  tx.type === 'income'
                                    ? 'bg-emerald-100 dark:bg-emerald-900/40 text-emerald-600 dark:text-emerald-400'
                                    : 'bg-red-100 dark:bg-red-900/40 text-red-600 dark:text-red-400'
                                }`}
                              >
                                {tx.type === 'income' ? (
                                  <ArrowUpRight className="w-4 h-4" />
                                ) : (
                                  <ArrowDownRight className="w-4 h-4" />
                                )}
                              </div>
                              <div className="min-w-0">
                                <p className="text-sm font-bold text-slate-800 dark:text-slate-100 truncate">
                                  {tx.title}
                                </p>
                                <p className="text-xs text-slate-500 dark:text-slate-400 truncate">
                                  {tx.subtitle} • {formatDateIndo(tx.date)}
                                </p>
                              </div>
                            </div>
                            <span
                              className={`text-sm font-extrabold shrink-0 ml-3 ${
                                tx.type === 'income'
                                  ? 'text-emerald-600 dark:text-emerald-400'
                                  : 'text-red-600 dark:text-red-400'
                              }`}
                            >
                              {tx.type === 'income' ? '+' : '-'}
                              {formatRupiah(tx.amount)}
                            </span>
                          </div>
                        ))}
                      </div>
                    )}
                  </div>

                  {/* Preview Target Tabungan */}
                  <div className="lg:col-span-5 p-6 rounded-2xl bg-white dark:bg-[#0F172A] border border-slate-200/80 dark:border-slate-800 shadow-sm flex flex-col justify-between">
                    <div>
                      <div className="flex items-center justify-between mb-5">
                        <div>
                          <h2 className="text-lg font-bold text-slate-900 dark:text-white">
                            Target Tabungan
                          </h2>
                          <p className="text-xs text-slate-500 dark:text-slate-400">
                            Progres barang impianmu
                          </p>
                        </div>
                        <button
                          onClick={() => setActivePage('savings')}
                          className="text-xs font-bold text-[#0284C7] dark:text-sky-400 hover:underline flex items-center gap-1"
                        >
                          Kelola <ChevronRight className="w-4 h-4" />
                        </button>
                      </div>

                      {savingsGoals.length === 0 ? (
                        <div className="text-center py-10 text-slate-400 text-sm">
                          Belum ada target tabungan. Buat impian pertamamu!
                        </div>
                      ) : (
                        <div className="space-y-4">
                          {savingsGoals.slice(0, 3).map((goal) => {
                            const pct = Math.min(
                              100,
                              Math.round((goal.currentAmount / goal.targetAmount) * 100)
                            );
                            return (
                              <div
                                key={goal.id}
                                className="p-4 rounded-xl bg-slate-50 dark:bg-slate-800/60 border border-slate-100 dark:border-slate-700/60"
                              >
                                <div className="flex items-center justify-between mb-1.5">
                                  <span className="text-sm font-bold text-slate-800 dark:text-slate-100">
                                    {goal.title}
                                  </span>
                                  <span className="text-xs font-extrabold text-[#0284C7] dark:text-sky-400">
                                    {pct}%
                                  </span>
                                </div>
                                <div className="w-full h-2.5 rounded-full bg-slate-200 dark:bg-slate-700 overflow-hidden">
                                  <div
                                    className={`h-full rounded-full transition-all duration-500 ${
                                      pct >= 100
                                        ? 'bg-emerald-500'
                                        : 'bg-gradient-to-r from-[#1E3A8A] to-[#38BDF8]'
                                    }`}
                                    style={{ width: `${pct}%` }}
                                  />
                                </div>
                                <div className="flex items-center justify-between text-[11px] text-slate-500 dark:text-slate-400 mt-2">
                                  <span>{formatRupiah(goal.currentAmount)}</span>
                                  <span>Target: {formatRupiah(goal.targetAmount)}</span>
                                </div>
                              </div>
                            );
                          })}
                        </div>
                      )}
                    </div>

                    <button
                      onClick={() => setActivePage('savings')}
                      className="mt-5 w-full py-3 rounded-xl bg-sky-50 dark:bg-slate-800 hover:bg-sky-100 dark:hover:bg-slate-700 text-[#1E3A8A] dark:text-sky-300 text-xs font-bold transition flex items-center justify-center gap-2"
                    >
                      <Target className="w-4 h-4" /> Tambah atau Setor Tabungan
                    </button>
                  </div>
                </div>
              </div>
            )}

            {/* =================================================================
                HALAMAN 2: CATATAN UANG SAKU (PEMASUKAN)
            ================================================================== */}
            {activePage === 'income' && (
              <div className="space-y-6">
                <div className="flex flex-col sm:flex-row sm:items-center sm:justify-between gap-2">
                  <div>
                    <h1 className="text-2xl font-extrabold text-slate-900 dark:text-white flex items-center gap-2">
                      <Wallet className="w-6 h-6 text-emerald-500" /> Catatan Uang Saku
                    </h1>
                    <p className="text-sm text-slate-500 dark:text-slate-400">
                      Catat setiap uang saku atau pemasukan agar saldo terpantau otomatis.
                    </p>
                  </div>
                  <div className="px-4 py-2 rounded-2xl bg-emerald-50 dark:bg-emerald-950/50 border border-emerald-200 dark:border-emerald-800">
                    <span className="text-xs text-emerald-700 dark:text-emerald-300 block">
                      Total Pemasukan Tercatat
                    </span>
                    <span className="text-lg font-extrabold text-emerald-600 dark:text-emerald-400">
                      {formatRupiah(totalAllIncome)}
                    </span>
                  </div>
                </div>

                {/* Form Tambah / Edit Pemasukan */}
                <form
                  onSubmit={handleSaveIncome}
                  className="p-6 rounded-2xl bg-white dark:bg-[#0F172A] border border-slate-200/80 dark:border-slate-800 shadow-sm space-y-4"
                >
                  <div className="flex items-center justify-between">
                    <h2 className="text-base font-bold text-slate-800 dark:text-white">
                      {editingIncomeId ? '✏️ Edit Catatan Uang Saku' : '➕ Tambah Uang Saku Baru'}
                    </h2>
                    {editingIncomeId && (
                      <button
                        type="button"
                        onClick={() => {
                          setEditingIncomeId(null);
                          setIncAmount('');
                          setIncNote('');
                        }}
                        className="text-xs text-slate-500 hover:text-red-500 font-semibold"
                      >
                        Batal Edit
                      </button>
                    )}
                  </div>

                  <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4">
                    <div>
                      <label className="block text-xs font-semibold text-slate-600 dark:text-slate-300 mb-1.5">
                        Nominal Uang (Rp) *
                      </label>
                      <input
                        type="number"
                        min="1"
                        required
                        placeholder="Contoh: 50000"
                        value={incAmount}
                        onChange={(e) => setIncAmount(e.target.value)}
                        className="w-full px-4 py-2.5 rounded-xl bg-slate-50 dark:bg-slate-800 border border-slate-200 dark:border-slate-700 text-sm focus:outline-none focus:ring-2 focus:ring-emerald-500"
                      />
                    </div>

                    <div>
                      <label className="block text-xs font-semibold text-slate-600 dark:text-slate-300 mb-1.5">
                        Sumber Pemasukan *
                      </label>
                      <select
                        value={incSource}
                        onChange={(e) => setIncSource(e.target.value)}
                        className="w-full px-4 py-2.5 rounded-xl bg-slate-50 dark:bg-slate-800 border border-slate-200 dark:border-slate-700 text-sm focus:outline-none focus:ring-2 focus:ring-emerald-500"
                      >
                        {INCOME_SOURCES.map((src) => (
                          <option key={src} value={src}>
                            {src}
                          </option>
                        ))}
                      </select>
                    </div>

                    <div>
                      <label className="block text-xs font-semibold text-slate-600 dark:text-slate-300 mb-1.5">
                        Tanggal Menerima *
                      </label>
                      <input
                        type="date"
                        required
                        value={incDate}
                        onChange={(e) => setIncDate(e.target.value)}
                        className="w-full px-4 py-2.5 rounded-xl bg-slate-50 dark:bg-slate-800 border border-slate-200 dark:border-slate-700 text-sm focus:outline-none focus:ring-2 focus:ring-emerald-500"
                      />
                    </div>

                    <div>
                      <label className="block text-xs font-semibold text-slate-600 dark:text-slate-300 mb-1.5">
                        Keterangan Tambahan
                      </label>
                      <input
                        type="text"
                        placeholder="Misal: Uang saku dari Ibu"
                        value={incNote}
                        onChange={(e) => setIncNote(e.target.value)}
                        className="w-full px-4 py-2.5 rounded-xl bg-slate-50 dark:bg-slate-800 border border-slate-200 dark:border-slate-700 text-sm focus:outline-none focus:ring-2 focus:ring-emerald-500"
                      />
                    </div>
                  </div>

                  <div className="flex justify-end pt-2">
                    <button
                      type="submit"
                      className="px-6 py-2.5 rounded-xl bg-emerald-600 hover:bg-emerald-700 text-white text-sm font-bold shadow-md shadow-emerald-600/20 transition flex items-center gap-2"
                    >
                      <Check className="w-4 h-4" />
                      {editingIncomeId ? 'Update Pemasukan' : 'Simpan Pemasukan'}
                    </button>
                  </div>
                </form>

                {/* Daftar Riwayat Pemasukan */}
                <div className="p-6 rounded-2xl bg-white dark:bg-[#0F172A] border border-slate-200/80 dark:border-slate-800 shadow-sm">
                  <h3 className="text-base font-bold text-slate-800 dark:text-white mb-4">
                    Daftar Uang Saku Masuk ({incomes.length})
                  </h3>

                  {incomes.length === 0 ? (
                    <p className="text-center py-8 text-sm text-slate-400">
                      Belum ada catatan pemasukan. Tambahkan uang sakumu di atas!
                    </p>
                  ) : (
                    <div className="space-y-3">
                      {[...incomes]
                        .sort((a, b) => b.date.localeCompare(a.date))
                        .map((item) => (
                          <div
                            key={item.id}
                            className="flex flex-col sm:flex-row sm:items-center justify-between gap-3 p-4 rounded-xl bg-slate-50 dark:bg-slate-800/60 border border-slate-100 dark:border-slate-700/60"
                          >
                            <div className="flex items-start gap-3">
                              <div className="p-2.5 rounded-xl bg-emerald-100 dark:bg-emerald-900/40 text-emerald-600 dark:text-emerald-400 mt-0.5">
                                <TrendingUp className="w-5 h-5" />
                              </div>
                              <div>
                                <div className="flex items-center gap-2 flex-wrap">
                                  <span className="font-bold text-sm text-slate-800 dark:text-white">
                                    {item.source}
                                  </span>
                                  <span className="text-[11px] px-2 py-0.5 rounded-full bg-slate-200/70 dark:bg-slate-700 text-slate-600 dark:text-slate-300">
                                    {formatDateIndo(item.date)}
                                  </span>
                                </div>
                                {item.note && (
                                  <p className="text-xs text-slate-500 dark:text-slate-400 mt-1">
                                    {item.note}
                                  </p>
                                )}
                              </div>
                            </div>

                            <div className="flex items-center justify-between sm:justify-end gap-4 pt-2 sm:pt-0 border-t sm:border-t-0 border-slate-200 dark:border-slate-700">
                              <span className="text-base font-extrabold text-emerald-600 dark:text-emerald-400">
                                +{formatRupiah(item.amount)}
                              </span>
                              <div className="flex items-center gap-1.5">
                                <button
                                  onClick={() => handleEditIncome(item)}
                                  title="Edit"
                                  className="p-2 rounded-lg bg-sky-100 dark:bg-sky-900/40 text-[#0284C7] dark:text-sky-400 hover:bg-sky-200 transition"
                                >
                                  <Edit3 className="w-4 h-4" />
                                </button>
                                <button
                                  onClick={() => handleDeleteIncome(item.id)}
                                  title="Hapus"
                                  className="p-2 rounded-lg bg-red-100 dark:bg-red-900/40 text-red-600 dark:text-red-400 hover:bg-red-200 transition"
                                >
                                  <Trash2 className="w-4 h-4" />
                                </button>
                              </div>
                            </div>
                          </div>
                        ))}
                    </div>
                  )}
                </div>
              </div>
            )}

            {/* =================================================================
                HALAMAN 3: CATATAN PENGELUARAN HARIAN
            ================================================================== */}
            {activePage === 'expense' && (
              <div className="space-y-6">
                <div className="flex flex-col sm:flex-row sm:items-center sm:justify-between gap-2">
                  <div>
                    <h1 className="text-2xl font-extrabold text-slate-900 dark:text-white flex items-center gap-2">
                      <ShoppingBag className="w-6 h-6 text-red-500" /> Catatan Pengeluaran Harian
                    </h1>
                    <p className="text-sm text-slate-500 dark:text-slate-400">
                      Catat jajanan, ongkos, dan keperluan sekolahmu setiap hari.
                    </p>
                  </div>
                  <div className="px-4 py-2 rounded-2xl bg-sky-50 dark:bg-sky-950/50 border border-sky-200 dark:border-sky-800">
                    <span className="text-xs text-sky-700 dark:text-sky-300 block">
                      Sisa Uang Saku Tersedia
                    </span>
                    <span className="text-lg font-extrabold text-[#1E3A8A] dark:text-sky-400">
                      {formatRupiah(currentBalance)}
                    </span>
                  </div>
                </div>

                {/* Peringatan Jika Saldo Tidak Mencukupi */}
                {insufficientBalanceAlert && (
                  <div className="p-4 rounded-2xl bg-red-50 dark:bg-red-950/70 border border-red-300 dark:border-red-800 flex items-start gap-3 text-red-800 dark:text-red-200">
                    <AlertTriangle className="w-5 h-5 text-red-600 shrink-0 mt-0.5" />
                    <div className="flex-1 text-sm">
                      <p className="font-bold">Peringatan: Uang Saku Tidak Mencukupi!</p>
                      <p className="text-xs mt-0.5">{insufficientBalanceAlert}</p>
                    </div>
                    <button
                      onClick={() => setInsufficientBalanceAlert(null)}
                      className="text-red-500 hover:text-red-700"
                    >
                      <X className="w-4 h-4" />
                    </button>
                  </div>
                )}

                {/* Form Tambah / Edit Pengeluaran */}
                <form
                  onSubmit={handleSaveExpense}
                  className="p-6 rounded-2xl bg-white dark:bg-[#0F172A] border border-slate-200/80 dark:border-slate-800 shadow-sm space-y-4"
                >
                  <div className="flex items-center justify-between">
                    <h2 className="text-base font-bold text-slate-800 dark:text-white">
                      {editingExpenseId ? '✏️ Edit Pengeluaran' : '➕ Catat Pengeluaran Baru'}
                    </h2>
                    {editingExpenseId && (
                      <button
                        type="button"
                        onClick={() => {
                          setEditingExpenseId(null);
                          setExpName('');
                          setExpAmount('');
                          setExpNote('');
                          setInsufficientBalanceAlert(null);
                        }}
                        className="text-xs text-slate-500 hover:text-red-500 font-semibold"
                      >
                        Batal Edit
                      </button>
                    )}
                  </div>

                  <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4">
                    <div>
                      <label className="block text-xs font-semibold text-slate-600 dark:text-slate-300 mb-1.5">
                        Nama Pengeluaran *
                      </label>
                      <input
                        type="text"
                        required
                        placeholder="Contoh: Nasi goreng kantin"
                        value={expName}
                        onChange={(e) => setExpName(e.target.value)}
                        className="w-full px-4 py-2.5 rounded-xl bg-slate-50 dark:bg-slate-800 border border-slate-200 dark:border-slate-700 text-sm focus:outline-none focus:ring-2 focus:ring-[#0284C7]"
                      />
                    </div>

                    <div>
                      <label className="block text-xs font-semibold text-slate-600 dark:text-slate-300 mb-1.5">
                        Nominal Pengeluaran (Rp) *
                      </label>
                      <input
                        type="number"
                        min="1"
                        required
                        placeholder="Contoh: 12000"
                        value={expAmount}
                        onChange={(e) => setExpAmount(e.target.value)}
                        className="w-full px-4 py-2.5 rounded-xl bg-slate-50 dark:bg-slate-800 border border-slate-200 dark:border-slate-700 text-sm focus:outline-none focus:ring-2 focus:ring-[#0284C7]"
                      />
                    </div>

                    <div>
                      <label className="block text-xs font-semibold text-slate-600 dark:text-slate-300 mb-1.5">
                        Kategori Pengeluaran *
                      </label>
                      <select
                        value={expCategory}
                        onChange={(e) => setExpCategory(e.target.value)}
                        className="w-full px-4 py-2.5 rounded-xl bg-slate-50 dark:bg-slate-800 border border-slate-200 dark:border-slate-700 text-sm focus:outline-none focus:ring-2 focus:ring-[#0284C7]"
                      >
                        {EXPENSE_CATEGORIES.map((cat) => (
                          <option key={cat.label} value={cat.label}>
                            {cat.label}
                          </option>
                        ))}
                      </select>
                    </div>

                    <div>
                      <label className="block text-xs font-semibold text-slate-600 dark:text-slate-300 mb-1.5">
                        Tanggal Transaksi *
                      </label>
                      <input
                        type="date"
                        required
                        value={expDate}
                        onChange={(e) => setExpDate(e.target.value)}
                        className="w-full px-4 py-2.5 rounded-xl bg-slate-50 dark:bg-slate-800 border border-slate-200 dark:border-slate-700 text-sm focus:outline-none focus:ring-2 focus:ring-[#0284C7]"
                      />
                    </div>

                    <div className="sm:col-span-2">
                      <label className="block text-xs font-semibold text-slate-600 dark:text-slate-300 mb-1.5">
                        Catatan Tambahan
                      </label>
                      <input
                        type="text"
                        placeholder="Misal: Beli bareng teman saat istirahat"
                        value={expNote}
                        onChange={(e) => setExpNote(e.target.value)}
                        className="w-full px-4 py-2.5 rounded-xl bg-slate-50 dark:bg-slate-800 border border-slate-200 dark:border-slate-700 text-sm focus:outline-none focus:ring-2 focus:ring-[#0284C7]"
                      />
                    </div>
                  </div>

                  <div className="flex justify-end pt-2">
                    <button
                      type="submit"
                      className="px-6 py-2.5 rounded-xl bg-gradient-to-r from-[#1E3A8A] to-[#0284C7] hover:opacity-95 text-white text-sm font-bold shadow-md shadow-sky-600/20 transition flex items-center gap-2"
                    >
                      <Check className="w-4 h-4" />
                      {editingExpenseId ? 'Update Pengeluaran' : 'Simpan Pengeluaran'}
                    </button>
                  </div>
                </form>

                {/* Daftar Pengeluaran */}
                <div className="p-6 rounded-2xl bg-white dark:bg-[#0F172A] border border-slate-200/80 dark:border-slate-800 shadow-sm">
                  <div className="flex items-center justify-between mb-4">
                    <h3 className="text-base font-bold text-slate-800 dark:text-white">
                      Riwayat Pengeluaran ({expenses.length})
                    </h3>
                    <span className="text-xs font-bold text-red-600 dark:text-red-400">
                      Total: {formatRupiah(totalAllExpense)}
                    </span>
                  </div>

                  {expenses.length === 0 ? (
                    <p className="text-center py-8 text-sm text-slate-400">
                      Belum ada pengeluaran tercatat. Keren, kamu sangat hemat!
                    </p>
                  ) : (
                    <div className="space-y-3">
                      {[...expenses]
                        .sort((a, b) => b.date.localeCompare(a.date))
                        .map((item) => (
                          <div
                            key={item.id}
                            className="flex flex-col sm:flex-row sm:items-center justify-between gap-3 p-4 rounded-xl bg-slate-50 dark:bg-slate-800/60 border border-slate-100 dark:border-slate-700/60"
                          >
                            <div className="flex items-start gap-3">
                              <div className="p-2.5 rounded-xl bg-red-100 dark:bg-red-900/40 text-red-600 dark:text-red-400 mt-0.5">
                                <TrendingDown className="w-5 h-5" />
                              </div>
                              <div>
                                <div className="flex items-center gap-2 flex-wrap">
                                  <span className="font-bold text-sm text-slate-800 dark:text-white">
                                    {item.name}
                                  </span>
                                  <span className="text-[11px] px-2.5 py-0.5 rounded-full bg-sky-100 dark:bg-sky-900/50 text-[#1E3A8A] dark:text-sky-300 font-medium">
                                    {item.category}
                                  </span>
                                  <span className="text-[11px] text-slate-400">
                                    • {formatDateIndo(item.date)}
                                  </span>
                                </div>
                                {item.note && (
                                  <p className="text-xs text-slate-500 dark:text-slate-400 mt-1">
                                    {item.note}
                                  </p>
                                )}
                              </div>
                            </div>

                            <div className="flex items-center justify-between sm:justify-end gap-4 pt-2 sm:pt-0 border-t sm:border-t-0 border-slate-200 dark:border-slate-700">
                              <span className="text-base font-extrabold text-red-600 dark:text-red-400">
                                -{formatRupiah(item.amount)}
                              </span>
                              <div className="flex items-center gap-1.5">
                                <button
                                  onClick={() => handleEditExpense(item)}
                                  title="Edit"
                                  className="p-2 rounded-lg bg-sky-100 dark:bg-sky-900/40 text-[#0284C7] dark:text-sky-400 hover:bg-sky-200 transition"
                                >
                                  <Edit3 className="w-4 h-4" />
                                </button>
                                <button
                                  onClick={() => handleDeleteExpense(item.id)}
                                  title="Hapus"
                                  className="p-2 rounded-lg bg-red-100 dark:bg-red-900/40 text-red-600 dark:text-red-400 hover:bg-red-200 transition"
                                >
                                  <Trash2 className="w-4 h-4" />
                                </button>
                              </div>
                            </div>
                          </div>
                        ))}
                    </div>
                  )}
                </div>
              </div>
            )}

            {/* =================================================================
                HALAMAN 4: SISA UANG & REKAP KEUANGAN
            ================================================================== */}
            {activePage === 'recap' && (
              <div className="space-y-6">
                <div>
                  <h1 className="text-2xl font-extrabold text-slate-900 dark:text-white flex items-center gap-2">
                    <FileSpreadsheet className="w-6 h-6 text-[#0284C7]" /> Sisa Uang & Rekap Keuangan
                  </h1>
                  <p className="text-sm text-slate-500 dark:text-slate-400">
                    Pantau kondisi keuanganmu berdasarkan rentang waktu pilihan.
                  </p>
                </div>

                {/* Filter Periode */}
                <div className="p-5 rounded-2xl bg-white dark:bg-[#0F172A] border border-slate-200/80 dark:border-slate-800 shadow-sm space-y-4">
                  <div className="flex flex-wrap items-center gap-2">
                    {[
                      { id: 'today', label: 'Hari ini' },
                      { id: 'week', label: 'Minggu ini' },
                      { id: 'month', label: 'Bulan ini' },
                      { id: 'custom', label: 'Pilih tanggal sendiri' }
                    ].map((f) => (
                      <button
                        key={f.id}
                        onClick={() => setRecapFilter(f.id as any)}
                        className={`px-4 py-2 rounded-xl text-xs sm:text-sm font-bold transition ${
                          recapFilter === f.id
                            ? 'bg-gradient-to-r from-[#1E3A8A] to-[#0284C7] text-white shadow-md'
                            : 'bg-slate-100 dark:bg-slate-800 text-slate-600 dark:text-slate-300 hover:bg-sky-50'
                        }`}
                      >
                        {f.label}
                      </button>
                    ))}
                  </div>

                  {recapFilter === 'custom' && (
                    <div className="grid grid-cols-1 sm:grid-cols-2 gap-4 pt-2 border-t border-slate-100 dark:border-slate-800">
                      <div>
                        <label className="block text-xs font-semibold text-slate-500 mb-1">
                          Dari Tanggal
                        </label>
                        <input
                          type="date"
                          value={customStartDate}
                          onChange={(e) => setCustomStartDate(e.target.value)}
                          className="w-full px-3.5 py-2 rounded-xl bg-slate-50 dark:bg-slate-800 border border-slate-200 dark:border-slate-700 text-sm"
                        />
                      </div>
                      <div>
                        <label className="block text-xs font-semibold text-slate-500 mb-1">
                          Sampai Tanggal
                        </label>
                        <input
                          type="date"
                          value={customEndDate}
                          onChange={(e) => setCustomEndDate(e.target.value)}
                          className="w-full px-3.5 py-2 rounded-xl bg-slate-50 dark:bg-slate-800 border border-slate-200 dark:border-slate-700 text-sm"
                        />
                      </div>
                    </div>
                  )}
                </div>

                {/* 4 Kartu Warna Rekap (Hijau Pemasukan, Merah Pengeluaran, Biru Saldo, Tabungan) */}
                <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4">
                  <div className="p-5 rounded-2xl bg-emerald-50 dark:bg-emerald-950/40 border border-emerald-200 dark:border-emerald-800/70">
                    <span className="text-xs font-bold text-emerald-700 dark:text-emerald-300">
                      Total Uang Saku Diterima
                    </span>
                    <p className="text-2xl font-extrabold text-emerald-600 dark:text-emerald-400 mt-1">
                      {formatRupiah(filteredRecapData.sumInc)}
                    </p>
                  </div>

                  <div className="p-5 rounded-2xl bg-red-50 dark:bg-red-950/40 border border-red-200 dark:border-red-800/70">
                    <span className="text-xs font-bold text-red-700 dark:text-red-300">
                      Total Pengeluaran
                    </span>
                    <p className="text-2xl font-extrabold text-red-600 dark:text-red-400 mt-1">
                      {formatRupiah(filteredRecapData.sumExp)}
                    </p>
                  </div>

                  <div className="p-5 rounded-2xl bg-sky-50 dark:bg-sky-950/40 border border-sky-200 dark:border-sky-800/70">
                    <span className="text-xs font-bold text-[#1E3A8A] dark:text-sky-300">
                      Sisa Uang (Periode Ini)
                    </span>
                    <p className="text-2xl font-extrabold text-[#0284C7] dark:text-sky-400 mt-1">
                      {formatRupiah(filteredRecapData.netBalance)}
                    </p>
                  </div>

                  <div className="p-5 rounded-2xl bg-indigo-50 dark:bg-indigo-950/40 border border-indigo-200 dark:border-indigo-800/70">
                    <span className="text-xs font-bold text-indigo-700 dark:text-indigo-300">
                      Total Berhasil Ditabung
                    </span>
                    <p className="text-2xl font-extrabold text-[#1E3A8A] dark:text-indigo-300 mt-1">
                      {formatRupiah(totalSavedAmount)}
                    </p>
                  </div>
                </div>

                {/* Rekap Harian Pemasukan & Pengeluaran */}
                <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
                  {/* Rekap Pengeluaran Harian */}
                  <div className="p-6 rounded-2xl bg-white dark:bg-[#0F172A] border border-slate-200/80 dark:border-slate-800 shadow-sm">
                    <h3 className="text-base font-bold text-slate-800 dark:text-white mb-4 flex items-center gap-2">
                      <span className="w-2.5 h-2.5 rounded-full bg-red-500" /> Rekap Pengeluaran Harian
                    </h3>
                    {filteredRecapData.dailyExpenseSummary.length === 0 ? (
                      <p className="text-xs text-slate-400 py-6 text-center">
                        Tidak ada pengeluaran pada periode ini.
                      </p>
                    ) : (
                      <div className="space-y-2.5">
                        {filteredRecapData.dailyExpenseSummary.map((d) => (
                          <div
                            key={d.date}
                            className="flex items-center justify-between p-3 rounded-xl bg-slate-50 dark:bg-slate-800/60 text-sm"
                          >
                            <span className="font-medium text-slate-700 dark:text-slate-300">
                              {formatDateIndo(d.date)}
                            </span>
                            <span className="font-bold text-red-600 dark:text-red-400">
                              -{formatRupiah(d.total)}
                            </span>
                          </div>
                        ))}
                      </div>
                    )}
                  </div>

                  {/* Rekap Pemasukan Harian */}
                  <div className="p-6 rounded-2xl bg-white dark:bg-[#0F172A] border border-slate-200/80 dark:border-slate-800 shadow-sm">
                    <h3 className="text-base font-bold text-slate-800 dark:text-white mb-4 flex items-center gap-2">
                      <span className="w-2.5 h-2.5 rounded-full bg-emerald-500" /> Rekap Pemasukan Harian
                    </h3>
                    {filteredRecapData.dailyIncomeSummary.length === 0 ? (
                      <p className="text-xs text-slate-400 py-6 text-center">
                        Tidak ada pemasukan pada periode ini.
                      </p>
                    ) : (
                      <div className="space-y-2.5">
                        {filteredRecapData.dailyIncomeSummary.map((d) => (
                          <div
                            key={d.date}
                            className="flex items-center justify-between p-3 rounded-xl bg-slate-50 dark:bg-slate-800/60 text-sm"
                          >
                            <span className="font-medium text-slate-700 dark:text-slate-300">
                              {formatDateIndo(d.date)}
                            </span>
                            <span className="font-bold text-emerald-600 dark:text-emerald-400">
                              +{formatRupiah(d.total)}
                            </span>
                          </div>
                        ))}
                      </div>
                    )}
                  </div>
                </div>

                {/* Riwayat Transaksi Gabungan */}
                <div className="p-6 rounded-2xl bg-white dark:bg-[#0F172A] border border-slate-200/80 dark:border-slate-800 shadow-sm">
                  <h3 className="text-base font-bold text-slate-800 dark:text-white mb-4">
                    Riwayat Transaksi Lengkap ({filteredRecapData.history.length})
                  </h3>
                  {filteredRecapData.history.length === 0 ? (
                    <p className="text-center py-8 text-sm text-slate-400">
                      Belum ada aktivitas transaksi pada rentang tanggal ini.
                    </p>
                  ) : (
                    <div className="divide-y divide-slate-100 dark:divide-slate-800">
                      {filteredRecapData.history.map((item) => (
                        <div
                          key={item.id}
                          className="py-3.5 flex items-center justify-between gap-4"
                        >
                          <div>
                            <p className="text-sm font-bold text-slate-800 dark:text-slate-100">
                              {item.title}
                            </p>
                            <p className="text-xs text-slate-500 dark:text-slate-400">
                              {item.categoryOrNote} • {formatDateIndo(item.date)}
                            </p>
                          </div>
                          <span
                            className={`text-sm font-extrabold ${
                              item.type === 'income'
                                ? 'text-emerald-600 dark:text-emerald-400'
                                : 'text-red-600 dark:text-red-400'
                            }`}
                          >
                            {item.type === 'income' ? '+' : '-'}
                            {formatRupiah(item.amount)}
                          </span>
                        </div>
                      ))}
                    </div>
                  )}
                </div>
              </div>
            )}

            {/* =================================================================
                HALAMAN 5: GRAFIK PENGELUARAN INTERAKTIF (RECHARTS)
            ================================================================== */}
            {activePage === 'charts' && (
              <div className="space-y-6">
                <div>
                  <h1 className="text-2xl font-extrabold text-slate-900 dark:text-white flex items-center gap-2">
                    <PieChartIcon className="w-6 h-6 text-[#0284C7]" /> Analisis & Grafik Keuangan
                  </h1>
                  <p className="text-sm text-slate-500 dark:text-slate-400">
                    Visualisasi otomatis dari setiap uang saku dan pengeluaran yang kamu catat.
                  </p>
                </div>

                {/* Banner Kesimpulan Otomatis Cerdas */}
                <div className="p-5 rounded-2xl bg-gradient-to-r from-[#0F172A] via-[#1E3A8A] to-[#0284C7] text-white shadow-lg flex items-start sm:items-center gap-4">
                  <div className="p-3 rounded-2xl bg-white/15 shrink-0">
                    <Sparkles className="w-6 h-6 text-sky-300" />
                  </div>
                  <div>
                    <span className="text-xs uppercase tracking-wider text-sky-200 font-bold">
                      Kesimpulan Pintar TrackOut
                    </span>
                    <p className="text-sm sm:text-base font-bold mt-0.5">
                      {chartAnalytics.topCategory
                        ? `Pengeluaran terbesarmu saat ini adalah ${
                            chartAnalytics.topCategory.fullLabel
                          } sebesar ${formatRupiah(chartAnalytics.topCategory.value)}.`
                        : 'Belum ada data pengeluaran. Mulai catat transaksimu untuk melihat analisis!'}
                    </p>
                  </div>
                </div>

                {/* 3 Kartu Statistik Sorotan Grafik */}
                <div className="grid grid-cols-1 sm:grid-cols-3 gap-4">
                  <div className="p-5 rounded-2xl bg-white dark:bg-[#0F172A] border border-slate-200/80 dark:border-slate-800 shadow-sm">
                    <span className="text-xs font-semibold text-slate-500 dark:text-slate-400">
                      Kategori Pengeluaran Terbanyak
                    </span>
                    <p className="text-base font-extrabold text-[#1E3A8A] dark:text-sky-400 mt-1">
                      {chartAnalytics.topCategory ? chartAnalytics.topCategory.fullLabel : '-'}
                    </p>
                    <p className="text-xs text-slate-500 mt-1">
                      {chartAnalytics.topCategory
                        ? formatRupiah(chartAnalytics.topCategory.value)
                        : 'Rp0'}
                    </p>
                  </div>

                  <div className="p-5 rounded-2xl bg-white dark:bg-[#0F172A] border border-slate-200/80 dark:border-slate-800 shadow-sm">
                    <span className="text-xs font-semibold text-slate-500 dark:text-slate-400">
                      Total Uang untuk Jajan (Makan & Minum)
                    </span>
                    <p className="text-xl font-extrabold text-amber-600 dark:text-amber-400 mt-1">
                      {formatRupiah(chartAnalytics.snackTotal)}
                    </p>
                    <p className="text-xs text-slate-500 mt-1">
                      {totalAllExpense > 0
                        ? `${Math.round((chartAnalytics.snackTotal / totalAllExpense) * 100)}% dari total pengeluaran`
                        : '0% dari total pengeluaran'}
                    </p>
                  </div>

                  <div className="p-5 rounded-2xl bg-white dark:bg-[#0F172A] border border-slate-200/80 dark:border-slate-800 shadow-sm">
                    <span className="text-xs font-semibold text-slate-500 dark:text-slate-400">
                      Rasio Sisa Uang Saku
                    </span>
                    <p className="text-xl font-extrabold text-emerald-600 dark:text-emerald-400 mt-1">
                      {totalAllIncome > 0
                        ? `${Math.max(0, Math.round((currentBalance / totalAllIncome) * 100))}%`
                        : '0%'}
                    </p>
                    <p className="text-xs text-slate-500 mt-1">
                      Sisa {formatRupiah(currentBalance)} dari {formatRupiah(totalAllIncome)}
                    </p>
                  </div>
                </div>

                {/* Grid 4 Grafik Recharts */}
                <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
                  {/* 1. Grafik Batang Pengeluaran Harian */}
                  <div className="p-6 rounded-2xl bg-white dark:bg-[#0F172A] border border-slate-200/80 dark:border-slate-800 shadow-sm">
                    <h3 className="text-base font-bold text-slate-800 dark:text-white mb-1">
                      1. Grafik Batang Pengeluaran Harian
                    </h3>
                    <p className="text-xs text-slate-500 dark:text-slate-400 mb-4">
                      Pengeluaranmu selama 7 hari terakhir
                    </p>
                    <div className="h-64 w-full">
                      <ResponsiveContainer width="100%" height="100%">
                        <BarChart data={chartAnalytics.dailyBarData}>
                          <CartesianGrid strokeDasharray="3 3" stroke="#94a3b8" opacity={0.2} />
                          <XAxis dataKey="tanggal" tick={{ fontSize: 12 }} />
                          <YAxis
                            tick={{ fontSize: 11 }}
                            tickFormatter={(val) => `${val / 1000}k`}
                          />
                          <Tooltip
                            formatter={(value: any) => [formatRupiah(Number(value)), 'Pengeluaran']}
                            contentStyle={{
                              borderRadius: '12px',
                              border: 'none',
                              boxShadow: '0 10px 25px -5px rgba(0,0,0,0.15)'
                            }}
                          />
                          <Bar dataKey="Pengeluaran" fill="#0284C7" radius={[8, 8, 0, 0]} />
                        </BarChart>
                      </ResponsiveContainer>
                    </div>
                  </div>

                  {/* 2. Grafik Lingkaran Kategori Pengeluaran */}
                  <div className="p-6 rounded-2xl bg-white dark:bg-[#0F172A] border border-slate-200/80 dark:border-slate-800 shadow-sm">
                    <h3 className="text-base font-bold text-slate-800 dark:text-white mb-1">
                      2. Grafik Lingkaran Kategori Pengeluaran
                    </h3>
                    <p className="text-xs text-slate-500 dark:text-slate-400 mb-4">
                      Proporsi pengeluaran berdasarkan kategori
                    </p>
                    {chartAnalytics.pieData.length === 0 ? (
                      <div className="h-64 flex items-center justify-center text-sm text-slate-400">
                        Belum ada data pengeluaran
                      </div>
                    ) : (
                      <div className="h-64 w-full">
                        <ResponsiveContainer width="100%" height="100%">
                          <PieChart>
                            <Pie
                              data={chartAnalytics.pieData}
                              dataKey="value"
                              nameKey="name"
                              cx="50%"
                              cy="50%"
                              innerRadius={52}
                              outerRadius={85}
                              paddingAngle={4}
                            >
                              {chartAnalytics.pieData.map((entry, idx) => (
                                <Cell key={`cell-${idx}`} fill={entry.color} />
                              ))}
                            </Pie>
                            <Tooltip
                              formatter={(value: any) => formatRupiah(Number(value))}
                              contentStyle={{
                                borderRadius: '12px',
                                border: 'none',
                                boxShadow: '0 10px 25px -5px rgba(0,0,0,0.15)'
                              }}
                            />
                            <Legend wrapperStyle={{ fontSize: '12px' }} />
                          </PieChart>
                        </ResponsiveContainer>
                      </div>
                    )}
                  </div>

                  {/* 3. Grafik Garis Tren Pengeluaran Bulanan */}
                  <div className="p-6 rounded-2xl bg-white dark:bg-[#0F172A] border border-slate-200/80 dark:border-slate-800 shadow-sm">
                    <h3 className="text-base font-bold text-slate-800 dark:text-white mb-1">
                      3. Grafik Garis Pengeluaran Bulanan
                    </h3>
                    <p className="text-xs text-slate-500 dark:text-slate-400 mb-4">
                      Tren pemasukan vs pengeluaran 6 bulan terakhir
                    </p>
                    <div className="h-64 w-full">
                      <ResponsiveContainer width="100%" height="100%">
                        <LineChart data={chartAnalytics.monthlyLineData}>
                          <CartesianGrid strokeDasharray="3 3" stroke="#94a3b8" opacity={0.2} />
                          <XAxis dataKey="bulan" tick={{ fontSize: 12 }} />
                          <YAxis
                            tick={{ fontSize: 11 }}
                            tickFormatter={(val) => `${val / 1000}k`}
                          />
                          <Tooltip
                            formatter={(value: any) => formatRupiah(Number(value))}
                            contentStyle={{
                              borderRadius: '12px',
                              border: 'none',
                              boxShadow: '0 10px 25px -5px rgba(0,0,0,0.15)'
                            }}
                          />
                          <Legend wrapperStyle={{ fontSize: '12px' }} />
                          <Line
                            type="monotone"
                            dataKey="Pemasukan"
                            stroke="#10B981"
                            strokeWidth={3}
                            dot={{ r: 4 }}
                          />
                          <Line
                            type="monotone"
                            dataKey="Pengeluaran"
                            stroke="#EF4444"
                            strokeWidth={3}
                            dot={{ r: 4 }}
                          />
                        </LineChart>
                      </ResponsiveContainer>
                    </div>
                  </div>

                  {/* 4. Grafik Perbandingan Pemasukan & Pengeluaran */}
                  <div className="p-6 rounded-2xl bg-white dark:bg-[#0F172A] border border-slate-200/80 dark:border-slate-800 shadow-sm">
                    <h3 className="text-base font-bold text-slate-800 dark:text-white mb-1">
                      4. Perbandingan Pemasukan & Pengeluaran
                    </h3>
                    <p className="text-xs text-slate-500 dark:text-slate-400 mb-4">
                      Total keseluruhan uang masuk vs uang keluar
                    </p>
                    <div className="h-64 w-full">
                      <ResponsiveContainer width="100%" height="100%">
                        <BarChart data={chartAnalytics.comparisonData} barGap={24}>
                          <CartesianGrid strokeDasharray="3 3" stroke="#94a3b8" opacity={0.2} />
                          <XAxis dataKey="name" tick={{ fontSize: 12 }} />
                          <YAxis
                            tick={{ fontSize: 11 }}
                            tickFormatter={(val) => `${val / 1000}k`}
                          />
                          <Tooltip
                            formatter={(value: any) => formatRupiah(Number(value))}
                            contentStyle={{
                              borderRadius: '12px',
                              border: 'none',
                              boxShadow: '0 10px 25px -5px rgba(0,0,0,0.15)'
                            }}
                          />
                          <Legend wrapperStyle={{ fontSize: '12px' }} />
                          <Bar dataKey="Pemasukan" fill="#10B981" radius={[10, 10, 0, 0]} barSize={60} />
                          <Bar dataKey="Pengeluaran" fill="#EF4444" radius={[10, 10, 0, 0]} barSize={60} />
                        </BarChart>
                      </ResponsiveContainer>
                    </div>
                  </div>
                </div>
              </div>
            )}

            {/* =================================================================
                HALAMAN 6: TARGET MENABUNG
            ================================================================== */}
            {activePage === 'savings' && (
              <div className="space-y-6">
                <div>
                  <h1 className="text-2xl font-extrabold text-slate-900 dark:text-white flex items-center gap-2">
                    <Target className="w-6 h-6 text-[#0284C7]" /> Target Menabung Pelajar
                  </h1>
                  <p className="text-sm text-slate-500 dark:text-slate-400">
                    Wujudkan barang impianmu dengan menyisihkan uang saku secara rutin!
                  </p>
                </div>

                {/* Form Buat Target Tabungan Baru */}
                <form
                  onSubmit={handleCreateGoal}
                  className="p-6 rounded-2xl bg-white dark:bg-[#0F172A] border border-slate-200/80 dark:border-slate-800 shadow-sm space-y-4"
                >
                  <h2 className="text-base font-bold text-slate-800 dark:text-white">
                    🎯 Buat Target Tabungan Baru
                  </h2>

                  <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4">
                    <div>
                      <label className="block text-xs font-semibold text-slate-600 dark:text-slate-300 mb-1.5">
                        Nama Barang / Tujuan *
                      </label>
                      <input
                        type="text"
                        required
                        placeholder="Misal: Sepatu sekolah baru"
                        value={savTitle}
                        onChange={(e) => setSavTitle(e.target.value)}
                        className="w-full px-4 py-2.5 rounded-xl bg-slate-50 dark:bg-slate-800 border border-slate-200 dark:border-slate-700 text-sm focus:outline-none focus:ring-2 focus:ring-[#0284C7]"
                      />
                    </div>

                    <div>
                      <label className="block text-xs font-semibold text-slate-600 dark:text-slate-300 mb-1.5">
                        Jumlah Target (Rp) *
                      </label>
                      <input
                        type="number"
                        min="1000"
                        required
                        placeholder="Misal: 350000"
                        value={savTarget}
                        onChange={(e) => setSavTarget(e.target.value)}
                        className="w-full px-4 py-2.5 rounded-xl bg-slate-50 dark:bg-slate-800 border border-slate-200 dark:border-slate-700 text-sm focus:outline-none focus:ring-2 focus:ring-[#0284C7]"
                      />
                    </div>

                    <div>
                      <label className="block text-xs font-semibold text-slate-600 dark:text-slate-300 mb-1.5">
                        Tabungan Awal (Rp)
                      </label>
                      <input
                        type="number"
                        min="0"
                        placeholder="Misal: 50000"
                        value={savInitial}
                        onChange={(e) => setSavInitial(e.target.value)}
                        className="w-full px-4 py-2.5 rounded-xl bg-slate-50 dark:bg-slate-800 border border-slate-200 dark:border-slate-700 text-sm focus:outline-none focus:ring-2 focus:ring-[#0284C7]"
                      />
                    </div>

                    <div>
                      <label className="block text-xs font-semibold text-slate-600 dark:text-slate-300 mb-1.5">
                        Batas Waktu Target *
                      </label>
                      <input
                        type="date"
                        required
                        value={savDeadline}
                        onChange={(e) => setSavDeadline(e.target.value)}
                        className="w-full px-4 py-2.5 rounded-xl bg-slate-50 dark:bg-slate-800 border border-slate-200 dark:border-slate-700 text-sm focus:outline-none focus:ring-2 focus:ring-[#0284C7]"
                      />
                    </div>
                  </div>

                  <div className="flex justify-end pt-2">
                    <button
                      type="submit"
                      className="px-6 py-2.5 rounded-xl bg-gradient-to-r from-[#1E3A8A] to-[#0284C7] hover:opacity-95 text-white text-sm font-bold shadow-md transition flex items-center gap-2"
                    >
                      <Plus className="w-4 h-4" /> Simpan Target
                    </button>
                  </div>
                </form>

                {/* Daftar Kartu Target Tabungan */}
                {savingsGoals.length === 0 ? (
                  <div className="p-10 rounded-2xl bg-white dark:bg-[#0F172A] border border-slate-200 dark:border-slate-800 text-center text-slate-400 text-sm">
                    Belum ada target menabung. Yuk tentukan impian pertamamu di atas!
                  </div>
                ) : (
                  <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
                    {savingsGoals.map((goal) => {
                      const pct = Math.min(
                        100,
                        Math.round((goal.currentAmount / goal.targetAmount) * 100)
                      );
                      const remaining = Math.max(0, goal.targetAmount - goal.currentAmount);
                      const isCompleted = goal.currentAmount >= goal.targetAmount;

                      return (
                        <div
                          key={goal.id}
                          className={`p-6 rounded-2xl border shadow-sm transition relative overflow-hidden ${
                            isCompleted
                              ? 'bg-gradient-to-br from-emerald-50 to-sky-50 dark:from-emerald-950/40 dark:to-slate-900 border-emerald-300 dark:border-emerald-700'
                              : 'bg-white dark:bg-[#0F172A] border-slate-200/80 dark:border-slate-800'
                          }`}
                        >
                          <div className="flex items-start justify-between gap-3 mb-4">
                            <div>
                              {isCompleted && (
                                <span className="inline-flex items-center gap-1 px-2.5 py-0.5 rounded-full bg-emerald-500 text-white text-[11px] font-bold mb-2">
                                  <Award className="w-3.5 h-3.5" /> Target Tercapai! 🎉
                                </span>
                              )}
                              <h3 className="text-lg font-extrabold text-slate-900 dark:text-white">
                                {goal.title}
                              </h3>
                              <p className="text-xs text-slate-500 dark:text-slate-400 flex items-center gap-1.5 mt-1">
                                <Calendar className="w-3.5 h-3.5" /> Batas waktu:{' '}
                                {formatDateIndo(goal.deadline)}
                              </p>
                            </div>
                            <button
                              onClick={() => handleDeleteGoal(goal.id)}
                              title="Hapus target"
                              className="p-2 rounded-xl text-slate-400 hover:text-red-500 hover:bg-red-50 dark:hover:bg-red-950/50 transition"
                            >
                              <Trash2 className="w-4 h-4" />
                            </button>
                          </div>

                          {/* Progress Bar */}
                          <div className="mb-4">
                            <div className="flex items-center justify-between text-xs font-bold mb-1.5">
                              <span className="text-slate-600 dark:text-slate-300">
                                Progres Pencapaian
                              </span>
                              <span
                                className={
                                  isCompleted
                                    ? 'text-emerald-600 dark:text-emerald-400'
                                    : 'text-[#0284C7] dark:text-sky-400'
                                }
                              >
                                {pct}%
                              </span>
                            </div>
                            <div className="w-full h-3.5 rounded-full bg-slate-200 dark:bg-slate-800 overflow-hidden">
                              <div
                                className={`h-full rounded-full transition-all duration-500 ${
                                  isCompleted
                                    ? 'bg-emerald-500'
                                    : 'bg-gradient-to-r from-[#1E3A8A] via-[#0284C7] to-[#38BDF8]'
                                }`}
                                style={{ width: `${pct}%` }}
                              />
                            </div>
                          </div>

                          {/* Detail Nominal */}
                          <div className="grid grid-cols-3 gap-2 p-3 rounded-xl bg-slate-50 dark:bg-slate-800/70 text-xs mb-4">
                            <div>
                              <span className="text-slate-400 block">Terkumpul</span>
                              <span className="font-extrabold text-emerald-600 dark:text-emerald-400">
                                {formatRupiah(goal.currentAmount)}
                              </span>
                            </div>
                            <div>
                              <span className="text-slate-400 block">Kurang</span>
                              <span className="font-extrabold text-slate-700 dark:text-slate-200">
                                {formatRupiah(remaining)}
                              </span>
                            </div>
                            <div>
                              <span className="text-slate-400 block">Target</span>
                              <span className="font-extrabold text-[#1E3A8A] dark:text-sky-400">
                                {formatRupiah(goal.targetAmount)}
                              </span>
                            </div>
                          </div>

                          {/* Tombol Tambah Tabungan */}
                          <button
                            onClick={() => {
                              setDepositModalGoal(goal);
                              setDepositAmount('');
                            }}
                            className={`w-full py-2.5 rounded-xl text-xs sm:text-sm font-bold transition flex items-center justify-center gap-2 ${
                              isCompleted
                                ? 'bg-emerald-600 hover:bg-emerald-700 text-white'
                                : 'bg-gradient-to-r from-[#1E3A8A] to-[#0284C7] hover:opacity-95 text-white shadow-md'
                            }`}
                          >
                            <PiggyBank className="w-4 h-4" /> Tambah Tabungan
                          </button>
                        </div>
                      );
                    })}
                  </div>
                )}

                {/* Modal Setor Tabungan */}
                {depositModalGoal && (
                  <div className="fixed inset-0 z-50 bg-slate-900/60 backdrop-blur-sm flex items-center justify-center p-4">
                    <div className="bg-white dark:bg-[#0F172A] border border-slate-200 dark:border-slate-800 rounded-3xl max-w-md w-full p-6 shadow-2xl space-y-4">
                      <div className="flex items-center justify-between">
                        <h3 className="text-lg font-extrabold text-slate-900 dark:text-white">
                          💰 Sisihkan Tabungan
                        </h3>
                        <button
                          onClick={() => setDepositModalGoal(null)}
                          className="p-1.5 rounded-xl text-slate-400 hover:bg-slate-100 dark:hover:bg-slate-800"
                        >
                          <X className="w-5 h-5" />
                        </button>
                      </div>

                      <p className="text-xs text-slate-500 dark:text-slate-400">
                        Target: <strong className="text-slate-800 dark:text-white">{depositModalGoal.title}</strong>
                        <br />
                        Sisa uang sakumu saat ini:{' '}
                        <strong className="text-[#0284C7] dark:text-sky-400">
                          {formatRupiah(currentBalance)}
                        </strong>
                      </p>

                      <form onSubmit={handleAddDeposit} className="space-y-4">
                        <div>
                          <label className="block text-xs font-semibold text-slate-600 dark:text-slate-300 mb-1.5">
                            Nominal yang Disisihkan (Rp)
                          </label>
                          <input
                            type="number"
                            min="500"
                            required
                            autoFocus
                            placeholder="Contoh: 20000"
                            value={depositAmount}
                            onChange={(e) => setDepositAmount(e.target.value)}
                            className="w-full px-4 py-3 rounded-xl bg-slate-50 dark:bg-slate-800 border border-slate-200 dark:border-slate-700 text-base font-bold focus:outline-none focus:ring-2 focus:ring-[#0284C7]"
                          />
                        </div>

                        <label className="flex items-center gap-2.5 text-xs text-slate-600 dark:text-slate-300 cursor-pointer select-none">
                          <input
                            type="checkbox"
                            checked={deductFromPocket}
                            onChange={(e) => setDeductFromPocket(e.target.checked)}
                            className="rounded text-[#0284C7] focus:ring-[#0284C7]"
                          />
                          <span>Potong otomatis dari Sisa Uang Saku saat ini</span>
                        </label>

                        <div className="flex gap-2.5 pt-2">
                          <button
                            type="button"
                            onClick={() => setDepositModalGoal(null)}
                            className="flex-1 py-2.5 rounded-xl bg-slate-100 dark:bg-slate-800 text-slate-600 dark:text-slate-300 text-sm font-bold"
                          >
                            Batal
                          </button>
                          <button
                            type="submit"
                            className="flex-1 py-2.5 rounded-xl bg-emerald-600 hover:bg-emerald-700 text-white text-sm font-bold shadow-md"
                          >
                            Simpan Tabungan
                          </button>
                        </div>
                      </form>
                    </div>
                  </div>
                )}
              </div>
            )}

            {/* =================================================================
                HALAMAN 7: TIPS HEMAT & TANTANGAN PELAJAR
            ================================================================== */}
            {activePage === 'tips' && (
              <div className="space-y-8">
                <div>
                  <h1 className="text-2xl font-extrabold text-slate-900 dark:text-white flex items-center gap-2">
                    <Lightbulb className="w-6 h-6 text-amber-500" /> Tips Hemat & Tantangan Pelajar
                  </h1>
                  <p className="text-sm text-slate-500 dark:text-slate-400">
                    Kebiasaan kecil setiap hari yang bikin tabunganmu cepat terkumpul!
                  </p>
                </div>

                {/* Tantangan Hemat Interaktif (Checklist) */}
                <div className="space-y-4">
                  <h2 className="text-lg font-bold text-slate-800 dark:text-white flex items-center gap-2">
                    🏆 Tantangan Hemat Mingguan (Checklist)
                  </h2>

                  <div className="grid grid-cols-1 md:grid-cols-3 gap-5">
                    {challenges.map((ch) => {
                      const completedCount = ch.days.filter(Boolean).length;
                      const isAllDone = completedCount === ch.days.length;

                      return (
                        <div
                          key={ch.id}
                          className={`p-5 rounded-2xl border shadow-sm transition flex flex-col justify-between ${
                            isAllDone
                              ? 'bg-emerald-50/70 dark:bg-emerald-950/30 border-emerald-300 dark:border-emerald-700'
                              : 'bg-white dark:bg-[#0F172A] border-slate-200/80 dark:border-slate-800'
                          }`}
                        >
                          <div>
                            <div className="flex items-center justify-between gap-2 mb-2">
                              <span className="text-xs font-extrabold px-2.5 py-1 rounded-full bg-sky-100 dark:bg-sky-900/50 text-[#1E3A8A] dark:text-sky-300">
                                {completedCount}/{ch.days.length} Hari
                              </span>
                              {isAllDone && (
                                <span className="text-xs font-bold text-emerald-600 dark:text-emerald-400">
                                  Selesai! 🎉
                                </span>
                              )}
                            </div>
                            <h3 className="text-base font-bold text-slate-900 dark:text-white">
                              {ch.title}
                            </h3>
                            <p className="text-xs text-slate-500 dark:text-slate-400 mt-1 leading-relaxed">
                              {ch.description}
                            </p>
                          </div>

                          <div className="mt-4 pt-4 border-t border-slate-100 dark:border-slate-800">
                            <div className="flex flex-wrap gap-2 mb-3">
                              {ch.days.map((done, idx) => (
                                <button
                                  key={idx}
                                  onClick={() => toggleChallengeDay(ch.id, idx)}
                                  className={`w-9 h-9 rounded-xl text-xs font-bold flex items-center justify-center transition ${
                                    done
                                      ? 'bg-emerald-500 text-white shadow-sm'
                                      : 'bg-slate-100 dark:bg-slate-800 text-slate-500 dark:text-slate-400 hover:bg-sky-100'
                                  }`}
                                >
                                  {done ? <Check className="w-4 h-4" /> : `H${idx + 1}`}
                                </button>
                              ))}
                            </div>
                            <p className="text-[11px] font-semibold text-[#0284C7] dark:text-sky-400">
                              ✨ Potensi hasil: {ch.rewardText}
                              {ch.dailyAmount
                                ? ` (Terkumpul: ${formatRupiah(completedCount * ch.dailyAmount)})`
                                : ''}
                            </p>
                          </div>
                        </div>
                      );
                    })}
                  </div>
                </div>

                {/* Daftar Kartu Tips Hemat */}
                <div className="space-y-4">
                  <h2 className="text-lg font-bold text-slate-800 dark:text-white">
                    💡 6 Jurus Jitu Mengatur Uang Saku SMA
                  </h2>
                  <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-5">
                    {HEMAT_TIPS.map((tip, idx) => (
                      <div
                        key={idx}
                        className="p-5 rounded-2xl bg-white dark:bg-[#0F172A] border border-slate-200/80 dark:border-slate-800 shadow-sm hover:border-sky-300 dark:hover:border-sky-700 transition"
                      >
                        <div className="text-3xl mb-3">{tip.icon}</div>
                        <h3 className="text-base font-bold text-slate-800 dark:text-white mb-1.5">
                          {tip.title}
                        </h3>
                        <p className="text-xs text-slate-500 dark:text-slate-400 leading-relaxed">
                          {tip.desc}
                        </p>
                      </div>
                    ))}
                  </div>
                </div>
              </div>
            )}

            {/* =================================================================
                HALAMAN 8: PENGATURAN
            ================================================================== */}
            {activePage === 'settings' && (
              <div className="space-y-6 max-w-3xl">
                <div>
                  <h1 className="text-2xl font-extrabold text-slate-900 dark:text-white flex items-center gap-2">
                    <Settings className="w-6 h-6 text-[#0284C7]" /> Pengaturan Aplikasi
                  </h1>
                  <p className="text-sm text-slate-500 dark:text-slate-400">
                    Sesuaikan profil pengguna, tampilan tema, dan data simulasi TrackOut.
                  </p>
                </div>

                {/* Form Profil Pelajar */}
                <form
                  onSubmit={handleSaveSettings}
                  className="p-6 rounded-2xl bg-white dark:bg-[#0F172A] border border-slate-200/80 dark:border-slate-800 shadow-sm space-y-4"
                >
                  <h2 className="text-base font-bold text-slate-800 dark:text-white">
                    👤 Profil Pengguna
                  </h2>
                  <div>
                    <label className="block text-xs font-semibold text-slate-600 dark:text-slate-300 mb-1.5">
                      Nama Pengguna
                    </label>
                    <input
                      type="text"
                      required
                      value={tempName}
                      onChange={(e) => setTempName(e.target.value)}
                      className="w-full px-4 py-2.5 rounded-xl bg-slate-50 dark:bg-slate-800 border border-slate-200 dark:border-slate-700 text-sm"
                    />
                  </div>
                  <div className="flex justify-end">
                    <button
                      type="submit"
                      className="px-5 py-2.5 rounded-xl bg-[#1E3A8A] hover:bg-[#0284C7] text-white text-xs sm:text-sm font-bold transition"
                    >
                      Simpan Profil
                    </button>
                  </div>
                </form>

                {/* Pengaturan Tema & Mata Uang */}
                <div className="p-6 rounded-2xl bg-white dark:bg-[#0F172A] border border-slate-200/80 dark:border-slate-800 shadow-sm space-y-4">
                  <h2 className="text-base font-bold text-slate-800 dark:text-white">
                    🎨 Preferensi Tampilan & Mata Uang
                  </h2>

                  <div className="flex items-center justify-between py-2">
                    <div>
                      <p className="text-sm font-semibold text-slate-800 dark:text-slate-100">
                        Tema Aplikasi (Terang / Gelap)
                      </p>
                      <p className="text-xs text-slate-500 dark:text-slate-400">
                        Pilih mode layar yang paling nyaman di matamu
                      </p>
                    </div>
                    <button
                      onClick={() => setProfile((p) => ({ ...p, darkMode: !p.darkMode }))}
                      className="px-4 py-2 rounded-xl bg-slate-100 dark:bg-slate-800 text-xs font-bold flex items-center gap-2 text-slate-800 dark:text-white"
                    >
                      {profile.darkMode ? (
                        <>
                          <Moon className="w-4 h-4 text-sky-400" /> Mode Gelap Aktif
                        </>
                      ) : (
                        <>
                          <Sun className="w-4 h-4 text-amber-500" /> Mode Terang Aktif
                        </>
                      )}
                    </button>
                  </div>

                  <div className="flex items-center justify-between py-2 border-t border-slate-100 dark:border-slate-800">
                    <div>
                      <p className="text-sm font-semibold text-slate-800 dark:text-slate-100">
                        Mata Uang Utama
                      </p>
                      <p className="text-xs text-slate-500 dark:text-slate-400">
                        Format pencatatan keuangan pelajar Indonesia
                      </p>
                    </div>
                    <span className="px-3.5 py-1.5 rounded-xl bg-emerald-50 dark:bg-emerald-950/50 text-emerald-700 dark:text-emerald-300 text-xs font-bold">
                      Rupiah (IDR - Rp)
                    </span>
                  </div>
                </div>

                {/* Reset Data Simulasi */}
                <div className="p-6 rounded-2xl bg-white dark:bg-[#0F172A] border border-red-200 dark:border-red-900/50 shadow-sm space-y-3">
                  <h2 className="text-base font-bold text-red-600 dark:text-red-400">
                    🔄 Reset Data Simulasi
                  </h2>
                  <p className="text-xs text-slate-500 dark:text-slate-400 leading-relaxed">
                    Ingin mengulang simulasi dari awal? Tombol ini akan menghapus catatanmu saat ini
                    dan mengembalikan contoh data uang saku & pengeluaran awal.
                  </p>

                  {!showResetConfirm ? (
                    <button
                      onClick={() => setShowResetConfirm(true)}
                      className="px-4 py-2.5 rounded-xl bg-red-50 dark:bg-red-950/60 hover:bg-red-100 text-red-600 dark:text-red-300 text-xs font-bold transition"
                    >
                      Reset ke Data Contoh Awal
                    </button>
                  ) : (
                    <div className="p-4 rounded-xl bg-red-50 dark:bg-red-950/80 border border-red-300 dark:border-red-800 flex flex-col sm:flex-row sm:items-center justify-between gap-3">
                      <span className="text-xs font-bold text-red-800 dark:text-red-200">
                        Yakin ingin mereset semua data transaksi & tabungan?
                      </span>
                      <div className="flex gap-2">
                        <button
                          onClick={() => setShowResetConfirm(false)}
                          className="px-3 py-1.5 rounded-lg bg-white dark:bg-slate-800 text-xs font-semibold"
                        >
                          Batal
                        </button>
                        <button
                          onClick={handleResetSimulation}
                          className="px-3 py-1.5 rounded-lg bg-red-600 text-white text-xs font-bold"
                        >
                          Ya, Reset Sekarang
                        </button>
                      </div>
                    </div>
                  )}
                </div>

                {/* Informasi Aplikasi TrackOut */}
                <div className="p-6 rounded-2xl bg-gradient-to-br from-[#0F172A] to-[#1E3A8A] text-white shadow-md space-y-3">
                  <TrackOutLogo size="md" lightText />
                  <p className="text-xs text-sky-100 leading-relaxed pt-1">
                    <strong>TrackOut v1.0</strong> adalah aplikasi simulasi pencatatan keuangan
                    pribadi yang dirancang khusus untuk pelajar SMA agar terbiasa mengatur uang
                    saku, mengendalikan pengeluaran jajan, dan mewujudkan target tabungan sejak dini.
                  </p>
                  <p className="text-[11px] text-sky-300 flex items-center gap-1.5">
                    <Info className="w-3.5 h-3.5" /> Bersifat simulasi edukatif & tersimpan aman di
                    browser perangkatmu (LocalStorage).
                  </p>
                </div>
              </div>
            )}
          </div>
        </main>

        {/* ================= BOTTOM NAVIGATION BAR (SMARTPHONE) ================= */}
        <nav className="lg:hidden fixed bottom-0 inset-x-0 z-40 bg-white/95 dark:bg-[#0F172A]/95 backdrop-blur-md border-t border-slate-200 dark:border-slate-800 px-2 py-1.5">
          <div className="grid grid-cols-5 gap-1">
            {navItems.slice(0, 4).map((item) => {
              const isActive = activePage === item.id;
              return (
                <button
                  key={item.id}
                  onClick={() => {
                    setActivePage(item.id);
                    setMobileMoreOpen(false);
                  }}
                  className={`flex flex-col items-center justify-center py-1.5 rounded-xl text-[10px] font-bold transition ${
                    isActive
                      ? 'text-[#0284C7] dark:text-sky-400 bg-sky-50 dark:bg-sky-950/60'
                      : 'text-slate-500 dark:text-slate-400'
                  }`}
                >
                  {item.icon}
                  <span className="mt-1 truncate max-w-[60px]">{item.shortLabel}</span>
                </button>
              );
            })}

            {/* Tombol Menu Lainnya untuk 4 Menu Sisa */}
            <button
              onClick={() => setMobileMoreOpen((prev) => !prev)}
              className={`flex flex-col items-center justify-center py-1.5 rounded-xl text-[10px] font-bold transition ${
                ['charts', 'savings', 'tips', 'settings'].includes(activePage) || mobileMoreOpen
                  ? 'text-[#0284C7] dark:text-sky-400 bg-sky-50 dark:bg-sky-950/60'
                  : 'text-slate-500 dark:text-slate-400'
              }`}
            >
              <Menu className="w-5 h-5" />
              <span className="mt-1">Lainnya</span>
            </button>
          </div>
        </nav>

        {/* Drawer Menu Lainnya di Mobile */}
        {mobileMoreOpen && (
          <div
            onClick={() => setMobileMoreOpen(false)}
            className="lg:hidden fixed inset-0 z-30 bg-slate-900/50 backdrop-blur-xs flex items-end pb-16"
          >
            <div
              onClick={(e) => e.stopPropagation()}
              className="w-full bg-white dark:bg-[#0F172A] rounded-t-3xl p-5 border-t border-slate-200 dark:border-slate-800 shadow-2xl space-y-2"
            >
              <div className="flex items-center justify-between mb-2">
                <span className="text-xs font-bold uppercase tracking-wider text-slate-400">
                  Menu TrackOut Lainnya
                </span>
                <button
                  onClick={() => setMobileMoreOpen(false)}
                  className="p-1 text-slate-400"
                >
                  <X className="w-4 h-4" />
                </button>
              </div>
              <div className="grid grid-cols-2 gap-2.5">
                {navItems.slice(4).map((item) => {
                  const isActive = activePage === item.id;
                  return (
                    <button
                      key={item.id}
                      onClick={() => {
                        setActivePage(item.id);
                        setMobileMoreOpen(false);
                      }}
                      className={`flex items-center gap-2.5 p-3.5 rounded-2xl text-xs font-bold transition ${
                        isActive
                          ? 'bg-gradient-to-r from-[#1E3A8A] to-[#0284C7] text-white'
                          : 'bg-slate-50 dark:bg-slate-800 text-slate-700 dark:text-slate-200'
                      }`}
                    >
                      {item.icon}
                      <span>{item.label}</span>
                    </button>
                  );
                })}
              </div>
            </div>
          </div>
        )}
      </div>
    </div>
  );
}
