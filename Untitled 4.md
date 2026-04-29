```'use client';

  

import React, { useState, useEffect } from 'react';

import { Pencil, Loader2, Save, ArrowRight, DollarSign, Euro, IndianRupee, Trash2, AlertCircle, Plus } from 'lucide-react';

import { getImportanceLevels, getProficiencyLevels, updateProficiencyLevels, updateJobtitleLevels, getJobTitleLevels, updateHeirarchy, updateSkillsHeirarchy } from '@/api/settings-api';

import { Modal } from '../(myorganization)/components/modal';

import { useToast } from '@/hooks/use-toast';

import Text from '../components/text';

import { useRouter } from 'next/navigation';

import { Card, CardContent } from "@/components/ui/card";

import { RadioGroup, RadioGroupItem } from "@/components/ui/radio-group";

import { Label } from "@/components/ui/label";

import { usePermissions } from '@/hooks/use-permissions';

import { Button } from '@/components/ui/button';

  

// Currency configuration

const CURRENCIES = [

{ code: 'USD', symbol: '$', name: 'US Dollar', icon: DollarSign },

{ code: 'EUR', symbol: '€', name: 'Euro', icon: Euro },

{ code: 'INR', symbol: '₹', name: 'Indian Rupee', icon: IndianRupee },

{ code: 'GBP', symbol: '£', name: 'British Pound', icon: null },

{ code: 'JPY', symbol: '¥', name: 'Japanese Yen', icon: null },

];

  

const DEFAULT_JT_LEVELS = [

{ level: 1, name: "Intern / Trainee", min_sal: 15000, max_sal: 30000, description: "Entry-level position learning fundamentals" },

{ level: 2, name: "Junior", min_sal: 30000, max_sal: 50000, description: "Developing skills with supervision" },

{ level: 3, name: "Professional", min_sal: 50000, max_sal: 80000, description: "Independent contributor with solid experience" },

{ level: 4, name: "Senior", min_sal: 80000, max_sal: 120000, description: "Expert with leadership responsibilities" },

{ level: 5, name: "Lead", min_sal: 120000, max_sal: 160000, description: "Guides teams and technical direction" },

{ level: 6, name: "Principal / Expert", min_sal: 160000, max_sal: 200000, description: "Strategic technical authority" },

{ level: 7, name: "Manager", min_sal: 200000, max_sal: 260000, description: "People management and team leadership" },

{ level: 8, name: "Senior Manager", min_sal: 260000, max_sal: 320000, description: "Multiple team oversight" },

{ level: 9, name: "Associate Director", min_sal: 320000, max_sal: 400000, description: "Department strategy execution" },

{ level: 10, name: "Director", min_sal: 400000, max_sal: 500000, description: "Department leadership and strategy" },

{ level: 11, name: "Vice President / Head", min_sal: 500000, max_sal: 650000, description: "Division leadership and P&L" },

{ level: 12, name: "Executive", min_sal: 650000, max_sal: 800000, description: "Senior organizational leadership" },

{ level: 13, name: "Senior Executive", min_sal: 800000, max_sal: 1000000, description: "Executive committee member" },

{ level: 14, name: "AVP / Business Head", min_sal: 1000000, max_sal: 1300000, description: "Business unit leadership" },

{ level: 15, name: "CXO", min_sal: 1300000, max_sal: 2000000, description: "C-level executive" }

];

  

// Character limit constants

const TITLE_MAX_LENGTH = 50;

const DESCRIPTION_MAX_LENGTH = 200;

const MAX_LEVELS = 15; // Maximum number of levels allowed

  
  

// --- SUB-COMPONENT: Level List Display ---

const LevelCard = ({ title, description, levels, isEditable, onEditClick }) => {

return (

<div className="bg-white p-6 rounded-xl shadow-sm border border-gray-200 h-full flex flex-col">

<div className="flex justify-between items-start border-b border-gray-100 pb-4 mb-4">

<div>

<h2 className="text-xl font-bold text-gray-900">{title}</h2>

<p className="text-sm text-gray-500 mt-1">{description}</p>

</div>

{isEditable && (

<button

onClick={onEditClick}

className="flex items-center gap-2 text-sm font-medium text-blue-600 bg-blue-50 hover:bg-blue-100 px-3 py-1.5 rounded-lg transition-colors"

>

<Pencil className="w-4 h-4" />

Edit

</button>

)}

</div>

  

<div className="flex-1 overflow-y-auto">

<ul className="space-y-3">

{levels.map((item, index) => (

<li key={index} className="p-3 bg-gray-50 rounded-lg border border-gray-100">

<div className="flex items-center justify-between mb-2">

<span className="text-gray-700 font-medium">{item.label}</span>

<div className="flex items-center gap-3">

<div

className={`w-8 h-8 rounded-full flex items-center justify-center text-[10px] font-bold shadow-sm ${item.color}`}

title={`Level ${item.level}`}

>

{item.label.split(' ').map(n => n[0]).join('')}

</div>

<span className="text-xs font-semibold text-gray-500 bg-white px-2 py-1 rounded border border-gray-200">

Lvl {item.level}

</span>

</div>

</div>

{item.description && (

<p className="text-xs text-gray-500 mt-1 line-clamp-2">{item.description}</p>

)}

</li>

))}

</ul>

</div>

</div>

);

};

  

const hierarchyOptions = [

{

id: "FULL",

title: "Full Hierarchy",

structure:

"Organization → Business Unit → Sub‑Business Unit → Department",

description:

"Best for large, complex organizations with multiple business divisions and regional subdivisions",

features: [

"Supports multi‑level business structure",

"Departments can belong to Organization (central), BU, or SBU",

"Example: Legal (Org‑level) | Sales (Healthcare BU) | Sales (Healthcare - Bangalore SBU)",

"Job Titles belong to Departments",

"Teams belong to Departments",

"Projects & Cohorts can belong to Org, BU, SBU, or Department",

],

bestFor:

"Best for: Large enterprises with complex regional operations",

},

{

id: "BU",

title: "Business Unit Hierarchy",

structure: "Organization → Business Unit → Department",

description:

"Ideal for mid to large organizations with distinct business divisions",

features: [

"Clear business unit structure",

"Departments can belong to Organization (central) or BU",

"Example: HR (Org‑level) | Engineering (Product BU) | Sales (Services BU)",

"Job Titles belong to Departments",

"Teams belong to Departments",

"Projects & Cohorts can belong to Org, BU, or Department",

],

bestFor:

"Best for: Mid-sized to large companies with clear business divisions",

},

{

id: "DEPARTMENT",

title: "Departmental Hierarchy",

structure: "Organization → Department",

description:

"Simple and straightforward for smaller organizations with functional departments",

features: [

"Simple, flat organizational structure",

"All departments belong directly to Organization",

"Example: Engineering | Sales | Marketing | HR | Finance",

"Job Titles belong to Departments",

"Teams belong to Departments",

"Projects & Cohorts can belong to Org or Department",

],

bestFor:

"Best for: Small to mid-sized companies with functional structure",

},

];

  

const skillHierarchyOptions = [

{

id: "4-level",

title: "4-Level Heirarchy",

structure:

"Domains → Knowledge Area → Skill Categories → Skills",

description:

"Most granular - ideal for Comprehensive skill management across diverse domains",

},

{

id: "3-level",

title: "3-Level Heirarchy",

structure: "Knowledge Area → Skill Categories → Skills",

description:

"Balanced approach-suitable for most organizations",

},

{

id: "2-level",

title: "2-Level Heirarchy",

structure: "Skill Categories → Skills",

description:

"Simplified - best for organizations with focused skill sets",

},

];

// --- SUB-COMPONENT: Job Title Level Row ---

const JobTitleLevelRow = ({ level, index, onUpdate, onDelete, currency, validationErrors }) => {

const currentCurrency = CURRENCIES.find(c => c.code === currency) || CURRENCIES[0];

const CurrencyIcon = currentCurrency.icon || DollarSign;

const hasError = validationErrors?.some(error => error.level === level.level);

  

return (

<div className={`grid grid-cols-12 gap-4 items-center p-4 bg-white rounded-lg border ${

hasError ? 'border-red-300 bg-red-50' : 'border-gray-200 hover:shadow-md'

} transition-shadow`}>

{/* Level Badge */}

<div className="col-span-1">

<span className={`inline-flex items-center justify-center w-10 h-10 rounded-full font-semibold text-sm border ${

hasError

? 'bg-red-100 text-red-700 border-red-300'

: 'bg-blue-50 text-blue-700 border-blue-200'

}`}>

L{level.level}

</span>

</div>

  

{/* Job Title Input */}

<div className="col-span-2">

<input

type="text"

value={level.name || ''}

onChange={(e) => {

const value = e.target.value.slice(0, TITLE_MAX_LENGTH);

onUpdate(index, 'name', value);

}}

className={`w-full px-3 py-2 text-sm border rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500 ${

hasError ? 'border-red-300' : 'border-gray-300'

}`}

placeholder="Job Title"

maxLength={TITLE_MAX_LENGTH}

/>

</div>

  

{/* Description Input */}

<div className="col-span-4">

<input

type="text"

value={level.description || ''}

onChange={(e) => {

const value = e.target.value.slice(0, DESCRIPTION_MAX_LENGTH);

onUpdate(index, 'description', value);

}}

className={`w-full px-3 py-2 text-sm border rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500 ${

hasError ? 'border-red-300' : 'border-gray-300'

}`}

placeholder="Description"

maxLength={DESCRIPTION_MAX_LENGTH}

/>

</div>

  

{/* Min Salary Input with Currency */}

<div className="col-span-2 relative">

<div className="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none">

<CurrencyIcon className="h-4 w-4 text-gray-400" />

</div>

<input

type="number"

value={level.min_sal}

onChange={(e) => onUpdate(index, 'min_sal', +e.target.value)}

className={`w-full pl-8 pr-3 py-2 text-sm border rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500 ${

hasError ? 'border-red-300' : 'border-gray-300'

}`}

placeholder="Min Salary"

min=""

step="1000"

/>

</div>

  

{/* Max Salary Input with Currency */}

<div className="col-span-2 relative">

<div className="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none">

<CurrencyIcon className="h-4 w-4 text-gray-400" />

</div>

<input

type="number"

value={level.max_sal}

onChange={(e) => onUpdate(index, 'max_sal', +e.target.value)}

className={`w-full pl-8 pr-3 py-2 text-sm border rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500 ${

hasError ? 'border-red-300' : 'border-gray-300'

}`}

placeholder="Max Salary"

min=""

step="1000"

/>

</div>

  

{/* Currency Display */}

<div className="col-span-1">

<span className="inline-flex items-center px-2.5 py-1 rounded-full text-xs font-medium bg-gray-100 text-gray-800">

{level.currency || currency}

</span>

</div>

  

{/* Delete Button */}

{/* <div className="col-span-1 flex justify-end">

<button

onClick={() => onDelete(level.level, level.uuid)}

className="p-2 text-gray-400 hover:text-red-600 hover:bg-red-50 rounded-lg transition-colors"

title="Delete this level"

>

<Trash2 className="w-5 h-5" />

</button>

</div> */}

</div>

);

};

  

// --- MAIN PAGE COMPONENT ---

const SettingsPage = () => {

// Data State

const [proficiencyLevels, setProficiencyLevels] = useState([]);

const [importanceLevels, setImportanceLevels] = useState([]);

const [jobTitleLevels, setJobTitleLevels] = useState([]);

const [isLoadingJT, setIsLoadingJT] = useState(true);

const router = useRouter();

  

const [selected, setSelected] = useState("full");

const [isEditMode, setIsEditMode] = useState(false);

const [showConfirmationModal, setShowConfirmationModal] = useState(false)

const [pendingAction, setPendingAction] = useState(null);

const { isFullHeirarchy, isBuOrFullHeirarchy, isDomainSkillHeirarchy, isKnowledgeSkillHeirarchy, isheirarchyChangeDisabled } = usePermissions();

const isBuHeirarchy = isBuOrFullHeirarchy();

const isSbuHeirarchy = isFullHeirarchy();

const isDomainHeirarchy = isDomainSkillHeirarchy();

const isKnowledgeHeirarchy = isKnowledgeSkillHeirarchy();

const isDisabled = isheirarchyChangeDisabled();

  

const [activeHierarchy, setActiveHierarchy] = useState();

const [activeSkillsHierarchy, setActiveSkillsHierarchy] = useState();

  

useEffect(() => {

if (isBuHeirarchy && isSbuHeirarchy) {

setActiveHierarchy("FULL");

} else if (isBuHeirarchy)

{

setActiveHierarchy("BU")

}

else

setActiveHierarchy("DEPARTMENT")

  

if (isDomainHeirarchy) {

setActiveSkillsHierarchy("4-level");

} else if (isKnowledgeHeirarchy)

{

setActiveSkillsHierarchy("3-level")

}

else

setActiveSkillsHierarchy("2-level")

  

}, [isBuHeirarchy, isSbuHeirarchy, isDomainHeirarchy, isKnowledgeHeirarchy]);

  
  
  

const handleChangeHierarchy = (id) => {

setActiveHierarchy(id)

}

  

const handleChangeSkillsHierarchy = (id) => {

setActiveSkillsHierarchy(id)

}

  

const handleSaveHeirarchy = (id) => {

setPendingAction({ type: 'ORG', id });

setShowConfirmationModal(true);

};

  

const handleSaveSkillsHeirarchy = (id) => {

setPendingAction({ type: 'SKILL', id });

setShowConfirmationModal(true);

};

  

const confirmSave = async () => {

if (!pendingAction) return;

setIsSaving(true);

try {

if (pendingAction.type === 'ORG') {

await updateHeirarchy(pendingAction.id);

} else if (pendingAction.type === 'SKILL') {

await updateSkillsHeirarchy(pendingAction.id);

}

setShowConfirmationModal(false);

window.location.reload();

} catch (error) {

console.error(`Failed to update ${pendingAction.type === 'ORG' ? 'organizational' : 'skill'} hierarchy:`, error);

toast({

title: "Error",

description: `Failed to update ${pendingAction.type === 'ORG' ? 'organizational' : 'skill'} hierarchy.`,

variant: "destructive",

});

} finally {

setIsSaving(false);

setPendingAction(null);

}

};

// UI State

const [isLoading, setIsLoading] = useState(true);

const [isSavingJT, setIsSavingJT] = useState(false);

const [isModalOpen, setIsModalOpen] = useState(false);

const [isSaving, setIsSaving] = useState(false);

const [currency, setCurrency] = useState("USD");

const [validationErrors, setValidationErrors] = useState([]);

// Edit State (Temporary state for the modal)

const [editingLevels, setEditingLevels] = useState([]);

  

const { toast } = useToast();

  

// 1. Fetch All Data

const fetchAllData = async () => {

try {

setIsLoading(true);

setIsLoadingJT(true);

const [profRes, impRes, jtRes] = await Promise.all([

getProficiencyLevels(),

getImportanceLevels(),

getJobTitleLevels()

]);

  

setProficiencyLevels(profRes?.proficiency_level?.levels || []);

setImportanceLevels(impRes?.importance_level?.levels || []);

// Process Job Title Levels from API

if (jtRes?.jt_levels && Array.isArray(jtRes.jt_levels)) {

// Sort by level number (convert string to number for comparison)

const sortedLevels = jtRes.jt_levels

.sort((a, b) => parseInt(a.level) - parseInt(b.level))

.map(item => ({

id: item.id,

uuid: item.uuid,

level: parseInt(item.level),

name: item.name || '',

description: item.description || '',

min_sal: item.min_sal !== null ? item.min_sal : "",

max_sal: item.max_sal !== null ? item.max_sal : "",

currency: item.currency || 'USD',

status: item.status

}));

setJobTitleLevels(sortedLevels);

// Set currency from first item if available

if (sortedLevels.length > 0) {

setCurrency(sortedLevels[0].currency);

}

} else {

// If no data from API, use defaults

setJobTitleLevels(DEFAULT_JT_LEVELS.map(j => ({ ...j, currency: "USD" })));

}

} catch (error) {

console.error("Failed to load settings:", error);

toast({

title: "Error",

description: "Failed to load settings. Using default values.",

variant: "destructive",

});

// Fallback to defaults on error

setJobTitleLevels(DEFAULT_JT_LEVELS.map(j => ({ ...j, currency: "USD" })));

} finally {

setIsLoading(false);

setIsLoadingJT(false);

}

};

  

useEffect(() => {

fetchAllData();

}, []);

  

// 2. Open Modal & Init Temp State

const handleOpenEdit = () => {

setEditingLevels(JSON.parse(JSON.stringify(proficiencyLevels)));

setIsModalOpen(true);

};

  

// 3. Handle Input Change in Modal

const handleLabelChange = (index, newLabel) => {

const updated = [...editingLevels];

updated[index].label = newLabel;

setEditingLevels(updated);

};

  

// 4. Save Proficiency Changes to API

const handleSave = async () => {

try {

setIsSaving(true);

const payload = {

proficiency_level: {

levels: editingLevels

}

};

  

await updateProficiencyLevels(payload);

await fetchAllData();

setIsModalOpen(false);

toast({

title: "Success",

description: "Proficiency levels updated successfully!",

});

} catch (error) {

console.error("Failed to save levels:", error);

toast({

title: "Error",

description: error.message || "Failed to update proficiency levels",

variant: "destructive",

});

} finally {

setIsSaving(false);

}

};

  

// 5. Handle Job Title Level Update

const handleJobTitleUpdate = (index, field, value) => {

const updated = [...jobTitleLevels];

updated[index] = { ...updated[index], [field]: value };

setJobTitleLevels(updated);

// Clear validation errors for this level when user starts editing

setValidationErrors(prev => prev.filter(error => error.level !== updated[index].level));

};

  

// 6. Handle Currency Change

const handleCurrencyChange = (e) => {

const newCurrency = e.target.value;

setCurrency(newCurrency);

setJobTitleLevels(prev =>

prev.map(level => ({ ...level, currency: newCurrency }))

);

};

  

// 7. Handle Delete Level

const handleDeleteLevel = (levelToDelete, uuid) => {

// Show confirmation dialog

if (window.confirm(`Are you sure you want to delete Level ${levelToDelete}? This action cannot be undone.`)) {

// If it's a new level (no uuid), just remove from state

// If it has uuid, you might want to call an API to delete it

const updatedLevels = jobTitleLevels.filter(level => level.level !== levelToDelete);

// Reassign level numbers to maintain sequence

const renumberedLevels = updatedLevels.map((level, index) => ({

...level,

level: index + 1

}));

setJobTitleLevels(renumberedLevels);

toast({

title: "Level Deleted",

description: `Level ${levelToDelete} has been removed.`,

});

}

};

  

// 8. Validate Job Title Levels

const validateJobTitles = () => {

const errors = [];

jobTitleLevels.forEach((level) => {

if (!level.name || level.name.trim() === '') {

errors.push({

level: level.level,

message: `Level ${level.level}: Title is required`

});

}

if (level.min_sal >= level.max_sal) {

errors.push({

level: level.level,

message: `Level ${level.level}: Max salary must be greater than min salary`

});

}

if (level.min_sal < 0 || level.max_sal < 0) {

errors.push({

level: level.level,

message: `Level ${level.level}: Salary cannot be negative`

});

}

});

  

setValidationErrors(errors);

return errors;

};

  

// 9. Save Job Title Levels to API

const handleSaveJobTitles = async () => {

// Validate first

const errors = validateJobTitles();

if (errors.length > 0) {

toast({

title: "Validation Error",

description: (

<div className="mt-2 space-y-1">

{errors.map((error, index) => (

<div key={index} className="text-sm text-red-600">

• {error.message}

</div>

))}

</div>

),

variant: "destructive",

});

return;

}

  

try {

setIsSavingJT(true);

// Prepare payload with title, description, and uuid if exists

const payload = {

jt_levels: jobTitleLevels.map(({ level, name, min_sal, max_sal, currency, description, uuid }) => {

const levelData = {

level,

name: name.trim(),

description: description?.trim() || '',

min_sal: min_sal,

max_sal: max_sal,

currency

};

// Include uuid if it exists (for existing records)

if (uuid) {

levelData.uuid = uuid;

}

return levelData;

})

};

  

await updateJobtitleLevels(payload);

// Refresh data to get updated uuids for new records

await fetchAllData();

toast({

title: "Success",

description: "Job title levels updated successfully!",

});

// Clear validation errors on successful save

setValidationErrors([]);

} catch (error) {

console.error("Failed to save job titles:", error);

toast({

title: "Error",

description: error.message || "Failed to update job title levels",

variant: "destructive",

});

} finally {

setIsSavingJT(false);

}

};

  

// 10. Reset Job Titles to Default

const handleResetJobTitles = () => {

if (window.confirm('Are you sure you want to reset to default values?')) {

setJobTitleLevels(DEFAULT_JT_LEVELS.map(j => ({ ...j, currency })));

setValidationErrors([]);

toast({

title: "Reset Complete",

description: "Job title levels reset to default values",

});

}

};

  

// 11. Add New Level

const handleAddNewLevel = () => {

if (jobTitleLevels.length >= MAX_LEVELS) {

toast({

title: "Maximum Levels Reached",

description: `You cannot add more than ${MAX_LEVELS} levels.`,

variant: "destructive",

});

return;

}

  

const newLevel = {

level: jobTitleLevels.length + 1,

name: "",

min_sal: "",

max_sal: "",

description: "",

currency: currency

// No uuid for new levels - will be added by backend

};

setJobTitleLevels([...jobTitleLevels, newLevel]);

};

  

// --- RENDER ---

if (isLoading && !proficiencyLevels.length) {

return (

<div className="flex h-screen items-center justify-center bg-[#F8F9FA]">

<Loader2 className="w-8 h-8 animate-spin text-blue-600" />

</div>

);

}

  

// Modal Footer

const modalFooter = (

<>

<button

onClick={() => setIsModalOpen(false)}

disabled={isSaving}

className="px-4 py-2 text-sm font-medium text-gray-700 bg-white border border-gray-300 rounded-lg hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-200"

>

Cancel

</button>

<button

onClick={handleSave}

disabled={isSaving}

className="flex items-center gap-2 px-4 py-2 text-sm font-medium text-white bg-blue-600 rounded-lg hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500 disabled:opacity-50"

>

{isSaving && <Loader2 className="w-4 h-4 animate-spin" />}

Save Changes

</button>

</>

);

  

// Confirmation Modal Footer

const confirmationModalFooter = (

<>

<button

onClick={() => setShowConfirmationModal(false)}

className="px-4 py-2 text-sm font-medium text-gray-700 bg-white border border-gray-300 rounded-lg hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-200"

>

Cancel

</button>

<button

onClick={confirmSave}

disabled={isSaving}

className="flex items-center gap-2 px-4 py-2 text-sm font-medium text-white bg-blue-600 rounded-lg hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500 disabled:opacity-50"

>

{isSaving && <Loader2 className="w-4 h-4 animate-spin" />}

Save Changes

</button>

</>

);

  

return (

<div className="min-h-screen bg-[#F8F9FA] p-8">

  

<div className="max-w-7xl mx-auto">

{/* Page Header */}

<header className="mb-8">

<h1 className="text-3xl font-bold text-[#141618]">System Configurations</h1>

<p className="text-gray-500 mt-2">Visual configuration for proficiency and importance levels used throughout the application.</p>

</header>

  

{/* Content Grid */}

<div className="space-y-6">

  

<div className="space-y-6">

<div>

<h2 className="text-xl font-semibold">

Organizational Structure Hierarchy

</h2>

<p className="text-sm text-muted-foreground mt-1">

Select the structure that best matches your organization. This will

determine how you organize departments, teams, and reporting lines.

</p>

</div>

<RadioGroup disabled={isDisabled} value={activeHierarchy} onValueChange={(val) => setActiveHierarchy(val)} className="space-y-2">

{hierarchyOptions.map((option) => (

<Label key={option.id} htmlFor={option.id}>

<Card onClick={() => !isDisabled && handleChangeHierarchy(option.id)}

className="cursor-pointer border rounded-xl transition-all hover:shadow-md data-[state=checked]:border-primary">

<CardContent className="p-6 space-y-4">

{/* Header */}

<div className="flex items-start gap-3">

<RadioGroupItem

value={option.id}

id={option.id}

className="mt-1"

checked={activeHierarchy === option.id}

disabled={isDisabled}

/>

  

<div className="space-y-1">

<h3 className="font-semibold text-base">

{option.title}

</h3>

  

<p className="text-sm font-medium text-muted-foreground">

{option.structure}

</p>

  

<p className="text-sm text-muted-foreground">

{option.description}

</p>

</div>

</div>

  

{/* Features */}

{/* <div className="space-y-2">

<p className="text-sm font-semibold">Key Features:</p>

  

<ul className="space-y-2">

{option.features.map((feature, i) => (

<li

key={i}

className="flex items-start gap-2 text-sm text-muted-foreground"

>

<CheckCircle2 className="h-4 w-4 mt-0.5 text-primary" />

<span>{feature}</span>

</li>

))}

</ul>

</div> */}

  

{/* Best For Box

<div className="bg-muted border rounded-lg px-4 py-3 text-sm font-medium text-primary">

{option.bestFor}

</div> */}

</CardContent>

</Card>

</Label>

))}

</RadioGroup>

<Button

onClick={() => handleSaveHeirarchy(activeHierarchy)}

disabled={isSaving && pendingAction?.type === 'ORG'}

>

{isSaving && pendingAction?.type === 'ORG' ? (

<Loader2 className="w-4 h-4 animate-spin mr-2" />

) : null}

Save Changes

</Button>

</div>

  

  

<div className="space-y-6">

<div>

<h2 className="text-xl font-semibold">

Skill Framework Hierarchy

</h2>

<p className="text-sm text-muted-foreground mt-1">

select how you want to organize and categorize skills in your organization.

</p>

</div>

  

<RadioGroup disabled={isDisabled} value={activeSkillsHierarchy} onValueChange={(val) => setActiveSkillsHierarchy(val)} className="space-y-2">

{skillHierarchyOptions.map((option) => (

<Label key={option.id} htmlFor={option.id}>

<Card onClick={() => !isDisabled && handleChangeSkillsHierarchy(option.id)}

className="cursor-pointer border rounded-xl transition-all hover:shadow-md data-[state=checked]:border-primary">

<CardContent className="p-6 space-y-4">

{/* Header */}

<div className="flex items-start gap-3">

<RadioGroupItem

value={option.id}

id={option.id}

className="mt-1"

checked={activeSkillsHierarchy === option.id}

disabled={isDisabled}

/>

  

<div className="space-y-1">

<h3 className="font-semibold text-base">

{option.title}

</h3>

  

<p className="text-sm font-medium text-muted-foreground">

{option.structure}

</p>

  

<p className="text-sm text-muted-foreground">

{option.description}

</p>

</div>

</div>

</CardContent>

</Card>

</Label>

))}

</RadioGroup>

<Button

onClick={() => handleSaveSkillsHeirarchy(activeSkillsHierarchy)}

disabled={isSaving && pendingAction?.type === 'SKILL'}

>

{isSaving && pendingAction?.type === 'SKILL' ? (

<Loader2 className="w-4 h-4 animate-spin mr-2" />

) : null}

Save Changes

</Button>

</div>

{/* Proficiency Levels (Editable) */}

<LevelCard

title="Proficiency Levels"

description="Define the expertise scale used for skill evaluation."

levels={proficiencyLevels}

isEditable={true}

onEditClick={handleOpenEdit}

/>

  

{/* Importance Levels (Read Only) */}

<LevelCard

title="Importance Levels"

description="Define the priority scale for tasks and requirements."

levels={importanceLevels}

isEditable={false}

/>

</div>

  

{/* Job Title Levels Section */}

<div className="bg-white rounded-xl shadow-sm border border-gray-200 p-6">

{/* Header */}

<div className="flex items-center justify-between mb-6">

<div>

<h2 className="text-3xl font-bold text-gray-900">Job Title Levels (L1-L15)</h2>

<p className="text-sm text-gray-500 mt-1">

Configure job title hierarchy, salary ranges, and descriptions

</p>

</div>

<div className="flex items-center gap-3">

{/* Currency Selector */}

<div className="flex items-center gap-2">

<span className="text-sm font-medium text-gray-700">Currency:</span>

<select

value={currency}

onChange={handleCurrencyChange}

className="px-3 py-2 text-sm border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500 bg-white"

>

{CURRENCIES.map(curr => (

<option key={curr.code} value={curr.code}>

{curr.symbol} {curr.code} - {curr.name}

</option>

))}

</select>

</div>

  

{/* Action Buttons */}

{/* <button

onClick={handleResetJobTitles}

className="px-4 py-2 text-sm font-medium text-gray-700 bg-white border border-gray-300 rounded-lg hover:bg-gray-50"

>

Reset

</button> */}

<button

onClick={handleSaveJobTitles}

disabled={isSavingJT || isLoadingJT}

className="flex items-center gap-2 px-4 py-2 text-sm font-medium text-white bg-blue-600 rounded-lg hover:bg-blue-700 disabled:opacity-50"

>

{isSavingJT ? (

<Loader2 className="w-4 h-4 animate-spin" />

) : (

<Save className="w-4 h-4" />

)}

Save Changes

</button>

</div>

</div>

  

{/* Column Headers */}

<div className="grid grid-cols-12 gap-4 mb-3 px-4 mt-10">

<div className="col-span-1 text-xs font-medium text-gray-500 uppercase tracking-wider">Level</div>

<div className="col-span-2 text-xs font-medium text-gray-500 uppercase tracking-wider">Job Title</div>

<div className="col-span-4 text-xs font-medium text-gray-500 uppercase tracking-wider">Description</div>

<div className="col-span-2 text-xs font-medium text-gray-500 uppercase tracking-wider">Min Salary</div>

<div className="col-span-2 text-xs font-medium text-gray-500 uppercase tracking-wider">Max Salary</div>

<div className="col-span-1 text-xs font-medium text-gray-500 uppercase tracking-wider">Currency</div>

<div className="col-span-1"></div> {/* Empty header for delete column */}

</div>

  

{/* Loading State for Job Titles */}

{isLoadingJT ? (

<div className="flex justify-center items-center py-12">

<Loader2 className="w-8 h-8 animate-spin text-blue-600" />

</div>

) : (

<>

{/* Job Title Rows */}

<div className="space-y-3">

{jobTitleLevels.map((jt, i) => (

<JobTitleLevelRow

key={jt.uuid || jt.level}

level={jt}

index={i}

onUpdate={handleJobTitleUpdate}

onDelete={handleDeleteLevel}

currency={currency}

validationErrors={validationErrors}

/>

))}

</div>

  

{/* Add Level Button and Summary Info */}

<div className="mt-6 flex flex-col items-start justify-between gap-y-4">

{/* Add New Level Button - Left side */}

{jobTitleLevels.length < MAX_LEVELS && (

<button

onClick={handleAddNewLevel}

className="flex items-center gap-2 px-4 py-2 text-sm font-medium text-blue-600 bg-blue-50 hover:bg-blue-100 rounded-lg transition-colors"

>

<Plus className="w-4 h-4" />

Add New Level

</button>

)}

  

{/* Summary Info - Right side */}

<div className="p-4 bg-blue-50 rounded-lg border border-blue-100 flex-1 ml-4">

<p className="text-sm text-blue-800">

<span className="font-semibold">Note:</span>

<span className="ml-1">Salary ranges should be progressive. Max salary should be greater than min salary for each level.</span>

<br />

<span className="text-xs text-blue-600 mt-1 block">

Title: {TITLE_MAX_LENGTH} chars max | Description: {DESCRIPTION_MAX_LENGTH} chars max | Max Levels: {MAX_LEVELS}

</span>

</p>

</div>

</div>

</>

)}

</div>

</div>

  

{/* Edit Modal */}

{!isDisabled && <div className="flex justify-between">

<div className="flex justify-start border-t">

<button

onClick={() =>

router.push(

"/getting-started"

)

}

className="inline-flex items-center border gap-2 px-6 py-3 bg-gray-100 text-gray-600 rounded-lg hover:bg-gray-200 transition"

>

Back to Setup hub

</button>

</div>

<div className="flex justify-end border-t">

<button

onClick={() =>

router.push(

"/structure-builder"

)

}

className="inline-flex items-center border gap-2 px-6 py-3 bg-gray-100 text-gray-600 rounded-lg hover:bg-gray-200 transition"

>

Skip and Continue

<ArrowRight className="w-4 h-4" />

</button>

<button

onClick={() =>

router.push(

"/structure-builder"

)

}

className="inline-flex items-center gap-2 px-6 py-3 bg-blue-600 text-white rounded-lg hover:bg-blue-700 transition"

>

Save and Continue

<ArrowRight className="w-4 h-4" />

</button>

</div>

</div>

}

  
  

{/* --- EDIT MODAL --- */}

<Modal

isOpen={isModalOpen}

onClose={() => setIsModalOpen(false)}

title="Edit Proficiency Labels"

subtitle="Update the display labels for each proficiency level."

modalMode="edit"

size="md"

alerttitle={false}

footer={modalFooter}

>

<div className="space-y-4">

{editingLevels.map((item, index) => (

<div key={item.level} className="grid grid-cols-12 gap-4 items-center">

<div className="col-span-3">

<span className={`inline-flex items-center px-3 py-1.5 text-sm font-semibold rounded-full border ${item.color}`}>

Level {item.level}

</span>

</div>

<div className="col-span-9">

<input

type="text"

value={item.label}

onChange={(e) => handleLabelChange(index, e.target.value)}

className="w-full px-3 py-2 text-sm border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500"

placeholder="Enter label name"

/>

</div>

</div>

))}

</div>

</Modal>

{/*--- CONFIRMATION MODAL --- */}

<Modal

isOpen={showConfirmationModal}

onClose={() => setShowConfirmationModal(false)}

title="Confirm Changes"

subtitle="Are you sure you want to save these changes?"

modalMode="edit"

size="md"

alerttitle={false}

footer={confirmationModalFooter}

>

<div className="space-y-4">

<p className="text-sm text-gray-700">

{pendingAction?.type === 'ORG'

? "This action will update the organizational structure hierarchy. This will affect how Business unit, Sub-Business Unit and Departments are organized."

: "This action will update the skill framework hierarchy. This might affect how skills are organized throughout the platform."

}

</p>

<p className="text-sm font-medium text-amber-600">

The page will reload after the changes are saved.

</p>

</div>

</Modal>

</div>

  
  

);

};

  

export default SettingsPage;
```

